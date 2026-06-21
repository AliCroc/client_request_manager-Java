# client_request_manager-Java
# SPRAWOZDANIE Z TWORZENIA ZADANIA NR 2
## Przedmiot: Programowanie Obiektowe
## Prowadzący: dr Marcin Gawilek
## Wykonał: Artur Ajchsztet, nr albumu: 143040

---

## 1. Wprowadzenie i Cele Projektu

### Cele zadania
Głównym celem zadania jest zaprojektowanie oraz implementacja systemu informatycznego służącego do odbioru, rejestracji oraz procesowania zapytań (zgłoszeń) od klientów. Projekt ma na celu praktyczne przetestowanie i weryfikację umiejętności poprawnego programowania obiektowego (OOP – Object-Oriented Programming) w języku Java, z zachowaniem dobrych praktyk inżynierii oprogramowania, takich jak enkapsulacja, dziedziczenie, polimorfizm oraz abstrakcja.

### Zamysł początkowy
Przedmiotem zadania jest wykonanie programu-systemu (ang. *Helpdesk / Ticketing System*), który działa w architekturze warstwowej (warstwa prezentacji, warstwa logiki biznesowej oraz warstwa danych). System umożliwia dwóm grupom użytkowników (Klientom oraz Użytkownikom Autoryzowanym: Operatorom i Administratorowi) interakcję z centralną bazą zapytań. Program rezygnuje z otwartej rejestracji ze względów bezpieczeństwa, przenosząc odpowiedzialność za zarządzanie personelem na jedno, predefiniowane konto Administratora.

---

## 2. Funkcje Programu (Wymagania Funkcjonalne)

Program realizuje następujące funkcjonalności podzielone według ról użytkowników:

### A. Funkcje ogólne i widok Klienta
1. **Odbiór zgłoszeń:** Możliwość złożenia nowego ZAPYTANIA przez klienta bez konieczności logowania.
2. **Podgląd statusu:** Klient po podaniu unikalnego numeru ID zapytania (ZUID) otrzymuje pełny wgląd w jego szczegóły (w tym treść odpowiedzi i aktualny status).

### B. System Użytkowników Autoryzowanych
1. **Brak rejestracji:** Dostęp do panelu administracyjnego i operatorskiego mają wyłącznie użytkownicy istniejący w bazie danych.
2. **Logowanie:** Autoryzacja oparta na parze login/hasło, zabezpieczona mechanizmem kryptograficznym.

### C. Uprawnienia OPERATORA
1. **Modyfikacja bazy zapytań:** Dodawanie lub zmiana treści odpowiedzi na zapytanie klienta.
2. **Zarządzanie statusem:** Zmiana statusu zapytania pomiędzy predefiniowanymi stanami: `NIEROZPOCZĘTY`, `ROZPOCZĘTY`, `OPÓŹNIONY`, `ZAMKNIĘTY`.
3. **Czyszczenie bazy:** Możliwość trwałego usunięcia z bazy danych zapytań, które posiadają status `ZAMKNIĘTY`.

### D. Uprawnienia ADMINISTRATORA (ADMIN)
1. **Wszystkie uprawnienia Operatora:** Administrator posiada pełny zestaw uprawnień modyfikacji zapytań.
2. **Konto wbudowane (Hardcoded):** Konto Administratora istnieje od pierwszego uruchomienia programu z niezmiennym loginem i stałym hasłem.
3. **Zarządzanie personelem:** Tworzenie nowych kont dla OPERATORÓW (wraz z generowaniem soli i haszowaniem ich haseł) oraz usuwanie istniejących OPERATORÓW z bazy.

### E. Interfejs i Dane
1. **Interfejs użytkownika:** Tekstowy interfejs graficzny realizowany w formie interaktywnego menu w konsoli (CLI).
2. **Trwałość danych:** Zapis struktur danych w bazie danych (np. SQLite / relacyjna baza danych w pamięci lub pliku), zapewniający integralność danych.

---

## 3. Architektura Systemu i Struktura Klas (OOP)

Program został zaprojektowany zgodnie z paradygmatem obiektowym. Poniżej znajduje się opis zaproponowanej struktury klas oraz relacji między nimi.

### Opis Klas w Programie

1. **`Main`**
   * *Rola:* Punkt wejściowy do aplikacji. Inicjalizuje bazę danych, ładuje konto Admina i uruchamia główną pętlę interfejsu.
2. **`Zapytanie`**
   * *Rola:* Klasa reprezentująca pojedyncze zgłoszenie klienta (encja danych).
   * *Atrybuty:* `zuid`, `imie`, `nazwisko`, `trescZapytania`, `trescOdpowiedzi`, `status` (jako Enum), `czasUtworzenia`, `czasOstatniejZmiany`.
3. **`Uzytkownik` (Klasa Abstrakcyjna)**
   * *Rola:* Klasa bazowa dla wszystkich autoryzowanych użytkowników systemu. Wprowadza wspólne cechy.
   * *Atrybuty:* `uuid`, `login`, `level` (Enum: `OPERATOR`, `ADMIN`).
4. **`Operator` (Dziedziczy po `Uzytkownik`)**
   * *Rola:* Reprezentuje pracownika obsługi klienta. Posiada metody do edycji zapytań.
5. **`Admin` (Dziedziczy po `Uzytkownik`)**
   * *Rola:* Reprezentuje administratora systemu. Rozszerza możliwości Operatora o zarządzanie kontami pracowników (`stworzOperatora()`, `usunOperatora()`).
6. **`DatabaseManager`**
   * *Rola:* Klasa odpowiedzialna za operacje wejścia/wyjścia na bazie danych (wzorzec DAO / CRUD). Izoluje logikę biznesową od bezpośrednich zapytań SQL.
7. **`ConsoleInterface`**
   * *Rola:* Odpowiada za wyświetlanie menu w konsoli, pobieranie danych od użytkownika (klasa `Scanner`) i przekazywanie ich do odpowiednich menedżerów.
8. **`AuthService`**
   * *Rola:* Odpowiada za proces logowania oraz operacje kryptograficzne (generowanie soli, haszowanie).

### Słownik Pojęć i Relacji OOP Zastosowanych w Projekcie

* **Klasa Abstrakcyjna (`Abstract Class`):** Klasa `Uzytkownik` jest oznaczona jako abstrakcyjna. Nie można utworzyć jej bezpośredniej instancji (obiektu). Służy jako wspólny szablon dla klas `Operator` i `Admin`.
* **Dziedziczenie (`Inheritance`):** Relacja typu **"jest czymś" (is-a)**. Klasy `Operator` oraz `Admin` dziedziczą po klasie `Uzytkownik`. Oznacza to, że przejmują jej pola (np. `login`) i metody, rozszerzając je o własne specyficzne zachowania.
* **Enkapsulacja / Hermetyzacja (`Encapsulation`):** Ukrywanie wewnętrznej implementacji obiektów. Wszystkie pola w klasie `Zapytanie` są prywatne (`private`). Dostęp do nich odbywa się wyłącznie za pomocą publicznych metod dostępowych (gettery i settery). Zapobiega to niekontrolowanej modyfikacji danych.
* **Polimorfizm (`Polymorphism`):** Zdolność obiektów różnych klas powiązanych dziedziczeniem do reagowania na to samo wywołanie metody w różny sposób. Może zostać wykorzystany np. przy sprawdzaniu uprawnień w menu systemu na podstawie polimorficznej referencji typu `Uzytkownik`.
* **Asocjacja (`Association`):** Ogólna relacja między klasami oznaczająca, że obiekty jednej klasy korzystają z obiektów drugiej. Klasa `ConsoleInterface` wchodzi w asocjację z `DatabaseManager` oraz `AuthService`, aby realizować polecenia użytkownika.
* **Kompozycja (`Composition`):** Silna relacja typu **"część całości" (has-a)**, gdzie cykl życia obiektu podrzędnego jest ściśle zależny od obiektu nadrzędnego. Na przykład baza danych (instancja w `DatabaseManager`) zawiera w sobie kolekcje obiektów `Zapytanie`.

---

## 4. Projekt Bazy Danych

System operuje na relacyjnej bazie danych składającej się z dwóch głównych tabel.

### Tabela: `Uzytkownicy`
Przechowuje dane niezbędne do uwierzytelniania i autoryzacji personelu.

| Nazwa Kolumny | Typ Danych | Ograniczenia (Constraints) | Opis |
| :--- | :--- | :--- | :--- |
| **UUID** | INT | PRIMARY KEY, AUTOINCREMENT | Unikalny identyfikator użytkownika |
| **login** | STRING / VARCHAR | UNIQUE, NOT NULL | Unikalna nazwa użytkownika do logowania |
| **hashedPassword** | STRING / VARCHAR | NOT NULL | Skrót hasła (wynik funkcji kryptograficznej) |
| **salt** | STRING / VARCHAR | NOT NULL | Losowy ciąg znaków unikalny dla każdego konta |
| **level** | ENUM / VARCHAR | NOT NULL | Poziom uprawnień: `OPERATOR` lub `ADMIN` |

### Tabela: `Zapytania`
Przechowuje dane dotyczące zgłoszeń od klientów oraz historię ich procesowania.

| Nazwa Kolumny | Typ Danych | Ograniczenia (Constraints) | Opis |
| :--- | :--- | :--- | :--- |
| **ZUID** | INT | PRIMARY KEY, AUTOINCREMENT | Unikalny identyfikator zapytania (numer zgłoszenia) |
| **imie** | STRING / VARCHAR | NOT NULL | Imię klienta zgłaszającego |
| **nazwisko** | STRING / VARCHAR | NOT NULL | Nazwisko klienta zgłaszającego |
| **tresc_zapytania** | TEXT / STRING | NOT NULL | Treść problemu/pytania opisanego przez klienta |
| **tresc_odpowiedzi**| TEXT / STRING | NULLABLE | Odpowiedź udzielona przez Operatora lub Admina |
| **status** | ENUM / VARCHAR | NOT NULL | Stan zgłoszenia: `NIEROZPOCZĘTY`, `ROZPOCZĘTY`, `OPÓŹNIONY`, `ZAMKNIĘTY` |
| **czas_utworzenia** | TIMESTAMP | NOT NULL | Data i godzina wpłynięcia zapytania do systemu |
| **data_zmiany_statusu**| TIMESTAMP | NOT NULL | Data i godzina ostatniej aktualizacji statusu zgłoszenia |

---

## 5. Logika Biznesowa i Bezpieczeństwo

### Autoryzacja konta ADMINISTRATORA
Konto `ADMIN` posiada status konta wbudowanego (*hardcoded*). Oznacza to, że jego dane logowania (login i hasło) są zdefiniowane bezpośrednio w kodzie aplikacji lub konfiguracji startowej. System podczas uruchomienia weryfikuje, czy konto to jest dostępne i uniemożliwia jego usunięcie. Admin nie przechodzi procesu standardowej rejestracji. Posiada on wyłączne prawo do wywoływania funkcji systemowej tworzącej obiekty klasy `Operator` i zapisywania ich struktur w bazie danych.

### Mechanizm Haszowania Haseł (Wysoki poziom ogólności)
W celu zapewnienia poufności danych personelu, system nie przechowuje haseł w postaci jawnego tekstu (ang. *plain text*). Zamiast tego zastosowano mechanizm haszowania z wykorzystaniem soli kryptograficznej (ang. *salting*):

1. **Rejestracja/Tworzenie Operatora przez Admina:**
   * Podczas tworzenia konta przez Administratora, system generuje unikalny, losowy ciąg znaków dla nowego użytkownika – tzw. **Sól (Salt)**.
   * Wprowadzone hasło tekstowe jest łączone (konkatenowane) z wygenerowaną solą.
   * Połączony ciąg jest poddawany jednostronnej funkcji skrótu (np. algorytm SHA-256). Wynikiem jest tzw. **Hasz (Hash)**.
   * W tabeli `Uzytkownicy` zapisywany jest login, wygenerowana sól oraz obliczony hasz.

2. **Proces Logowania:**
   * Użytkownik podaje w konsoli login i hasło.
   * System wyszukuje w bazie rekord o podanym loginie i pobiera przypisaną do niego sól oraz zapisany hasz.
   * System łączy wpisane przez użytkownika hasło z pobraną z bazy solą i ponownie uruchamia funkcję skrótu.
   * Jeżeli nowo obliczony hasz jest identyczny z haszem zapisanym w bazie danych, użytkownik zostaje pomyślnie zalogowany.

Zastosowanie soli zapobiega atakom metodą słownikową oraz przy użyciu tablic tęczowych (ang. *Rainbow Tables*), uniemożliwiając odczytanie hasła nawet w przypadku nieautoryzowanego dostępu do bazy danych.