# Planer Nauki - raport z heurystyk Nielsena

## Opis projektu

Projekt przedstawia prosty interfejs aplikacji webowej **Planer Nauki** napisanej w Vue.js. Aplikacja nie wymaga backendu i zapisuje dane w `localStorage`, dzięki czemu po odświeżeniu strony użytkownik nadal widzi swoje plany.

Interfejs zawiera trzy widoki:

- **Dashboard** - szybki przegląd liczby planów, przedmiotów, łącznego czasu nauki i najbliższego terminu.
- **Dodaj plan** - formularz tworzenia planu nauki.
- **Podsumowanie** - lista zapisanych planów z możliwością usuwania i cofnięcia ostatniej operacji.

## Zastosowane heurystyki

1. **Widoczność statusu systemu**
   - Nad treścią aplikacji pojawia się pasek komunikatu informujący o zapisaniu planu, błędach formularza, usunięciu wpisu lub cofnięciu operacji.
   - Aktywna zakładka w nawigacji pokazuje, w którym widoku znajduje się użytkownik.

2. **Zgodność systemu z rzeczywistością**
   - Etykiety formularza używają naturalnego języka: "Przedmiot", "Temat nauki", "Termin", "Czas nauki", "Priorytet".
   - Dashboard pokazuje informacje w sposób znany z codziennego planowania: liczba planów, łączny czas i najbliższy termin.

3. **Kontrola i swoboda użytkownika**
   - Formularz ma przycisk "Anuluj", który czyści dane i wraca do dashboardu.
   - Formularz nie usuwa danych, w przypadku kiedy użytkownik przez przypadek zmieni zakładkę.
   - Po usunięciu planu pojawia się możliwość "Cofnij", dzięki czemu użytkownik może naprawić przypadkowe usunięcie.

4. **Zapobieganie błędom**
   - Formularz blokuje zapis pustych pól.
   - Pole daty nie pozwala wybrać terminu z przeszłości.
   - Czas nauki musi mieć co najmniej 15 minut.

5. **Spójność i standardy**
   - Wszystkie widoki mają ten sam nagłówek, nawigację i styl.
   - Przyciski główne, drugorzędne i niebezpieczne operacje mają powtarzalne kolory oraz wygląd.

6. **Rozpoznawanie zamiast przypominania**
   - Formularz zawiera placeholdery, np. "np. Matematyka" i "np. Pochodne funkcji".
   - Widok dodawania planu zawiera podpowiedzi, które pomagają wypełnić formularz bez zapamiętywania zasad.

7. **Elastyczność i efektywność użycia**
   - Dashboard ma szybkie akcje "Dodaj plan" oraz "Zobacz podsumowanie".
   - Gdy nie ma danych, użytkownik może jednym kliknięciem dodać przykładowe plany.

8. **Estetyka i minimalizm**
   - Interfejs pokazuje tylko informacje potrzebne w danym widoku.
   - Layout wykorzystuje proste karty, ograniczoną paletę kolorów i czytelną hierarchię treści.

9. **Pomoc w rozpoznawaniu i naprawie błędów**
   - Komunikaty pod polami formularza mówią konkretnie, co należy poprawić, np. "Wpisz nazwę przedmiotu" albo "Termin nie może być z przeszłości".
   - Pasek komunikatu informuje, że plan nie został zapisany i wskazuje pola oznaczone komunikatem.

10. **Pomoc i dokumentacja**
    - Sekcja "Podpowiedzi" w formularzu pełni rolę krótkiej pomocy kontekstowej.

## Rozwiązane problemy projektowe

- Użytkownik zawsze wie, gdzie jest w aplikacji dzięki aktywnej zakładce i komunikatom po wykonanych akcjach.
- Formularz nie pozwala zapisać niepełnych lub nielogicznych danych.
- Przypadkowe usunięcie planu można cofnąć.
- Puste stany są opisane prostym językiem i proponują następną akcję.
- Widoki są spójne wizualnie, dlatego aplikacja jest łatwa do zrozumienia mimo prostego prototypu.
