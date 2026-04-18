## Zadanie 1.1: Zastosowania grafiki

- Graficzny interfejs użytkownika (GUI).
	- Rola grafiki komputerowej w tej dziedzinie polega na stworzeniu wizualnego środowiska pracy (opartego na oknach, ikonach, menu i przyciskach), dzięki któremu komputer staje się użytecznym narzędziem dla przeciętnego użytkownika. Interfejs graficzny eliminuje konieczność korzystania wyłącznie z linii poleceń, sprawiając, że obsługa systemu jest intuicyjna i dostępna nie tylko dla specjalistów.
- Poligrafia i skład drukarski (DTP).
	- W tej dziedzinie rola grafiki komputerowej polega na zastąpieniu tradycyjnych metod typograficznych systemami cyfrowymi, służącymi do przygotowania publikacji. Umożliwia ona obróbkę tekstu i obrazów oraz łączenie ich w jedną, zamierzoną formę gotową do druku. Dodatkowo grafika komputerowa pozwala na precyzyjne dostosowanie barw do specyfiki wykorzystywanych urządzeń drukujących.
- Symulacja i wirtualna rzeczywistość.
	- Rola grafiki komputerowej w symulacjach polega na generowaniu skomplikowanych, realistycznych wizualizacji w czasie rzeczywistym, co jest kluczowe dla zapewnienia użytkownikowi wiarygodnych wrażeń wzrokowych. Dzięki temu możliwe jest prowadzenie bezpiecznych i tanich szkoleń obsługi drogiego sprzętu (np. samolotów czy maszyn wojskowych) w dowolnych warunkach. Systemy oparte na grafice pozwalają na przećwiczenie sytuacji niestandardowych, które byłyby niemożliwe do odtworzenia w rzeczywistości.

## Zadanie 1.2: Porównanie modeli grafiki

| Cecha                              | Grafika wektorowa                                                         | Grafika rastrowa                                                    |
| ---------------------------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Sposób zapamiętania <br>obrazu** | Sekwencyjna struktura <br>danych obrazowych                               | Obraz zapamiętywany <br>w postaci mapy                              |
| **Zajętość pamięci**               | Zależna od <br>skomplikowania obrazu                                      | Zajętość pamięci niezależna od skomplikowania obrazu                |
| **Skalowalność**                   | Możliwość skalowania                                                      | Brak możliwości skalowania (bez utraty jakości)                     |
| **Odbiór rysunku (jakość linii)**  | Odbiór rysunku niezależny<br>od urządzenia <br>(linia jest zawsze gładka) | Odbiór zależny od urządzenia<br>(linia widoczna w postaci schodków) |
## Zadanie 1.3: Studium przypadku (dobór modelu)
- **Scenariusz A:** Projektujesz logo firmy, które musi być użyte na małej wizytówce oraz na wielkim banerze reklamowym.
	- **Wybór:** **Grafika wektorowa** 
	- **Uzasadnienie:** Kluczowym czynnikiem jest tutaj **skalowalność**. Musimy mieć dwa zupełnie różne rozmiary. Grafika wektorowa pozwala właśnie na skalowanie bez strat. Ponadto ważna jest **odbiór rysunku** – w przypadku dużego formatu banera w grafice wektorowej linia będzie „zawsze gładka”, podczas gdy w grafice rastrowej po powiększeniu widoczne byłyby piksele w postaci „schodków”, co wyglądałoby nieestetycznie.
- **Scenariusz B**: Obróbka zdjęcia z wakacji
	- **Wybór:** **Grafika rastrowa**
	- **Uzasadnienie:** Fotografia to obraz zawierający wiele szczegółów i kolorów, idealnie nadający się do **zapamiętywania obrazu** w grafice rastrowej, czyli „w postaci mapy” (mapy pikseli). Operacje na „plamie” i edycja kolorów wymagają pracy z konkretnymi punktami tej mapy, właśnie możliwość manipulowania każdym pikselem osobno pozwala edytować obiekty na obrazie, a dzięki statycznej pamięci nie zmienia się rozmiar obrazu.

## Zadanie 1.4: Analiza formatów plików

 ## 1. Formaty nieskalowalne (rastrowe):
- **JPG (JPEG):**
    - **Paleta i kompresja:** Format ten wykorzystuje **pełną paletę barw** oraz stosuje **stratną kompresję DCT**.
    - **Główna zaleta:** Jego zaletą jest **bardzo wydajna kompresja**.
- **PNG:**
    - **Kompresja:** Stosuje **wydajniejszą kompresję bezstratną**.
    - **Paleta i dodatkowe zalety:** Posiada **pełną paletę barw**. Dwie dodatkowe zalety to **obsługa kanału Alfa** (przezroczystość) oraz **brak ograniczeń licencyjnych**.
### 2. Formaty skalowalne (wektorowe):
- **PS (Postscript):**
    - **Opis:** Jest to **język opisu strony**.
    - **Twórca:** Został opracowany przez firmę **Adobe**.
- **SVG:**
    - **Podstawa standardu:** Standard ten został opracowany w oparciu o **XML**.
    - **Przeznaczenie:** Został stworzony **na potrzeby WWW** (stron internetowych).