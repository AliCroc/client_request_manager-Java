### 1. Dziedziczenie (Inheritance)
* **W projekcie:** Relacja między klasą `<Użytkownik>` a klasami `Operator` i `Admin`. Na diagramie oznaczona jest strzałką z pustym grotem (Generalizacja) skierowaną od klas podrzędnych do klasy nadrzędnej.
* **Znaczenie:** Klasy `Operator` i `Admin` są wyspecjalizowanymi wersjami `Użytkownika`. Przejmują one wszystkie jego właściwości (`id`, `login`) oraz metody (`dodajOdpowiedz()`, `zmienStatusOdpowiedzi()`), dzięki czemu unikamy dublowania kodu.

### 2. Klasa Abstrakcyjna (Abstract Class)
* **W projekcie:** Klasa `<Użytkownik>` (zapisana w nawiasach ostrokątnych lub kursywą).
* **Znaczenie:** Nie możemy utworzyć "czystego" obiektu typu `Użytkownik` (operacja `new Użytkownik()` jest zablokowana). Klasa ta istnieje wyłącznie po to, aby stanowić wspólny mianownik i kontrakt dla rzeczywistych obiektów, jakimi są `Operator` i `Admin`.

### 3. Hermetyzacja / Enkapsulacja (Encapsulation)
* **W projekcie:** Widoczność pól oznaczona modyfikatorami `+` (public) oraz `-` (private). Na przykład pola `- imie` czy `- nazwisko` w klasie `Zapytanie` są prywatne, natomiast metody w modułach są publiczne.
* **Znaczenie:** Ukrywanie wewnętrznego stanu obiektu przed bezpośrednią modyfikacją z zewnątrz. Dostęp do wrażliwych danych odbywa się kontrolowanie – wyłącznie przez wystawione metody publiczne.

### 4. Typ Wyliczeniowy (Enum / Enumerate)
* **W projekcie:** Struktura `<<Enumerate>> Status`.
* **Znaczenie:** Ograniczenie wartości pola `status` w klasie `Zapytanie` wyłącznie do czterech z góry zdefiniowanych opcji. Zapewnia to odporność programu na błędy typu literówki (np. wpisanie "zakonczone" zamiast "ZAKOŃCZONY").

### 5. Wzorzec Projektowy Singleton
* **W projekcie:** Klasa `PowiernikBazyDanych` posiada prywatne pole `- instancja: PowiernikBazyDanych` oraz publiczną metodę statyczną `+ getInstancja()`.
* **Znaczenie:** Gwarantuje, że w całej aplikacji będzie istniał tylko jeden, globalny obiekt odpowiedzialny za zarządzanie pamięcią bazy danych. Każdy moduł (np. `ModułLogowania`) korzysta dokładnie z tej samej instancji.

### 6. Relacje Zależności i Asocjacji (Asocjacja, Zależność)
* **Asocjacja (Posiadanie/Używanie):** `ModułLogowania` posiada pole `powiernikBazy: PowiernikBazyDanych` oraz `zalogowany: Użytkownik`. Oznacza to, że obiekty te ściśle ze sobą współpracują i komunikują się za pomocą komunikatów (metod).
* **Zależność (Dependency):** Oznaczona przerywaną strzałką między `ŁącznikBazyDanych` a `Baza danych SQL`. Klasa "wypytuje" i "otrzymuje" dane, co oznacza, że zmiana w strukturze bazy danych SQL wpłynie na implementację metod w klasie łącznika.

---

## 5. Projekt Baz Danych

Struktura tabel odpowiada obiektom domenowym z diagramu klas.

### Tabela: `Uzytkownicy`
| Nazwa Kolumny | Typ Danych | Ograniczenia | Opis |
| :--- | :--- | :--- | :--- |
| **UUID** | INT | PRIMARY KEY, AUTOINCREMENT | Mapowane na `id` w klasie `Użytkownik` |
| **login** | VARCHAR | UNIQUE, NOT NULL | Nazwa użytkownika |
| **hashedPassword**| VARCHAR | NOT NULL | Skrót hasła zabezpieczony solą |
| **salt** | VARCHAR | NOT NULL | Sól kryptograficzna |
| **level** | VARCHAR | NOT NULL | Poziom dostępu (`OPERATOR`, `ADMIN`) |

### Tabela: `Zapytania`
| Nazwa Kolumny | Typ Danych | Ograniczenia | Opis |
| :--- | :--- | :--- | :--- |
| **ZUID** | INT | PRIMARY KEY, AUTOINCREMENT | Mapowane na `id_zapytania` |
| **czas_utworzenia**| INT / TIMESTAMP| NOT NULL | Czas rejestracji zgłoszenia |
| **imie** | VARCHAR | NOT NULL | Imię klienta |
| **nazwisko** | VARCHAR | NOT NULL | Nazwisko klienta |
| **tresc** | TEXT | NOT NULL | Treść pytania |
| **odpowiedz** | TEXT | NULLABLE | Treść udzielonej odpowiedzi |
| **status** | VARCHAR | NOT NULL | Wartość z enum: `NIEROZPOCZĘTY` itp. |
| **data_zmiany** | INT / TIMESTAMP| NOT NULL | Ostatnia modyfikacja statusu |

---

## 6. Logika Bezpieczeństwa (Haszowanie i Sól)

Aplikacja implementuje bezpieczne przechowywanie danych uwierzytelniających:
1. **Konto Administratora (ADMIN):** Jest wpisane na stałe w kodzie systemu (*hardcoded*). Uruchamia ono aplikację i pozwala na dostęp do metod `dodajOperatora()` oraz `usuńOperatora()`.
2. **Generowanie haseł Operatorów:** Podczas wywołania `dodajOperatora(login, haslo)`, system generuje unikalną dla każdego rekordu wartość tekstową (sól). Następnie łączy jawne hasło z solą i przetwarza je jednostronną funkcją skrótu (np. SHA-256).
3. **Weryfikacja:** Podczas logowania w `ModułLogowania`, wpisane hasło jest łączone z solą pobraną z bazy dla danego loginu. Jeśli wygenerowany hasz zgadza się z rekordem w tabeli `Uzytkownicy`, obiekt typu `Użytkownik` zostaje przypisany do zmiennej sesyjnej `zalogowany`.