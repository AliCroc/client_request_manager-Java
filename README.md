# client_request_manager-Java
# SPRAWOZDANIE Z TWORZENIA ZADANIA NR 2
## Przedmiot: Programowanie Obiektowe
## Prowadzący: dr Marcin Gawilek
## Wykonał: Artur Ajchsztet, nr albumu: 143040

---

## 1. Wprowadzenie i Cele Projektu

### Cele zadania
Głównym celem zadania jest zaprojektowanie oraz implementacja systemu informatycznego służącego do odbioru, rejestracji oraz procesowania zapytań i zgłoszeń od klientów. Projekt ma na celu praktyczne przetestowanie i weryfikację umiejętności poprawnego programowania obiektowego (OOP) w języku Java, z zachowaniem paradygmatów enkapsulacji, dziedziczenie, polimorfizmu oraz abstrakcji.

### Zamysł początkowy
System działa w architekturze warstwowej z wyraźnym podziałem na klasy danych, moduły logiczne/sterujące oraz warstwę dostępu do baz danych. Program umożliwia klientom zgłaszanie spraw oraz interakcję pracownikom (Operatorom i Administratorowi) z bazą zgłoszeń za pośrednictwem tekstowego menu konsoli.

---

## 2. Funkcje Programu (Wymagania Funkcjonalne)

Program realizuje następujące funkcjonalności:
* **Odbiór i podgląd zgłoszeń (Klient):** Możliwość utworzenia zapytania, sprawdzenia jego stanu po ID oraz dodania komentarza.
* **Autoryzacja (Pracownicy):** Logowanie do systemu bez publicznej rejestracji.
* **Zarządzanie zapytaniami (Operator/Admin):** Dodawanie odpowiedzi, modyfikacja statusów, usuwanie komentarzy oraz czyszczenie bazy z zamkniętych spraw.
* **Zarządzanie personelem (Admin):** Wyłączne uprawnienie do tworzenia i usuwania kont Operatorów.

---

## 3. Architektura i Opis Klas (Zgodnie ze Schematem UML)

Aplikacja została podzielona na logiczne moduły (ramki strukturalne):

### A. Klasy Danowe (Encje)
* **`Zapytanie`** – reprezentuje pojedyncze zgłoszenie. Zawiera unikalne ID, metadane czasowe, dane klienta, treść, odpowiedź oraz status.
* **`Status` (Typ Wyliczeniowy - Enum)** – definiuje skończony zbiór stanów zapytania: `NIEROZPOCZĘTY`, `ROZPOCZĘTY`, `OPÓŹNIONY`, `ZAKOŃCZONY`.
* **`Użytkownik` (Klasa Abstrakcyjna)** – klasa bazowa przechowująca wspólny identyfikator (`id`) oraz `login`, a także deklarująca wspólne metody wykonawcze dla personelu.
* **`Operator`** – klasa pochodna, reprezentująca standardowego pracownika z konstruktorem inicjalizującym.
* **`Admin`** – klasa pochodna, rozszerzająca uprawnienia o metody `dodajOperatora()` oraz `usuńOperatora()`.

### B. Moduły Logiczne i Interfejs
* **`Menu`** – główny komponent interfejsu użytkownika, agregujący `ModułKlienta` i sterujący aplikacją.
* **`ModułKlienta`** – odpowiada za interakcję z klientem (tworzenie, sprawdzanie zapytania, dodawanie komentarza).
* **`ModułLogowania`** – zarządza sesją użytkownika (przechowuje obiekt aktualnie zalogowanego `Użytkownika`) i odpowiada za autoryzację.

### C. Warstwa Bazy Danych
* **`PowiernikBazyDanych`** – centralny punkt zarządzania danymi w aplikacji, zaimplementowany jako **Singleton**. Odpowiada za dostarczanie list użytkowników i zapytań do logiki biznesowej.
* **`ŁącznikBazyDanych`** – interfejs lub klasa abstrakcyjna definiująca metody dostępowe do struktur bazodanowych.
* **`Baza danych SQL`** – zewnętrzny silnik bazy danych, z którym komunikuje się aplikacja.

---

## 4. Pojęcia i Relacje OOP w Projekcie (Analiza Diagramu)

Przesłany schemat klas w pełni realizuje kluczowe założenia programowania obiektowego. Oto jak poszczególne koncepcje mapują się na strukturę programu: