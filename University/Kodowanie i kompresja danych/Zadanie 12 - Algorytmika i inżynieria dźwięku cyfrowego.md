## **Zadanie 1.1: Cyfryzacja sygnału ciągłego**

**1. Błąd kwantyzacji:** to różnica między ciągłą wartością sygnału a jego cyfrową reprezentacją wynikającą ze skończonej liczby bitów. Zwiększenie częstotliwości próbkowania (np. do 96 kHz) **nie eliminuje** tego błędu, ponieważ błąd ten zależy od rozdzielczości bitowej (zbioru poziomów kwantyzacji $U$), a nie od gęstości próbek w czasie. Jak wskazuje literatura (Derpich i in., 2006), optymalizacja jakości sygnału wymaga balansu między obydwoma parametrami, a samo zwiększenie częstotliwości próbkowania bez zwiększenia liczby bitów może jedynie zmienić charakterystykę szumu, ale nie usunie niedokładności wynikającej z kwantyzacji.
**Literatura:** [(PDF) Quantization and Sampling of Not Necessarily Band-Limited Signals](https://www.researchgate.net/publication/224641533_Quantization_and_Sampling_of_Not_Necessarily_Band-Limited_Signals)

**2. Aliasing:** Filtr dolnoprzepustowy  jest niezbędny przed przetwornikiem A/C, aby usunąć z sygnału analogowego wszystkie częstotliwości przekraczające częstotliwość Nyquista ($f_{Nyquist} = \frac{1}{2} f_{sample}$). Bez tego filtra, składowe o wysokich częstotliwościach zostałyby błędnie zinterpretowane jako niskie częstotliwości (aliasing), zanieczyszczając sygnał użyteczny.
Analiza przypadku (25 kHz przy próbkowaniu 44.1 kHz):
Zgodnie z twierdzeniem Nyquista-Shannona, maksymalna częstotliwość, którą można poprawnie zarejestrować przy próbkowaniu 44.1 kHz, wynosi 22.05 kHz.
1. Ponieważ 25 kHz > 22.05 kHz, wystąpi zjawisko **aliasingu**.
2. Dane nie zostaną zapisane poprawnie; zamiast sygnału 25 kHz, w systemie pojawi się „alias” (fałszywy obraz) o niższej częstotliwości (ok. 19.1 kHz).
3. [home.strw.leidenuniv.nl/~por/AOT2019/docs/AOT_2019_Ex13_NyquistTheorem.pdf](https://home.strw.leidenuniv.nl/~por/AOT2019/docs/AOT_2019_Ex13_NyquistTheorem.pdf)

## **Zadanie 1.2: Złożoność obliczeniowa a architektura systemu**

11. System wbudowany: Wybiorę kodek **FLAC**. W systemach wbudowanych o niskim poborze mocy kluczowa jest wydajność dekodowania. Jak wskazują autorzy artykułu, FLAC został zaprojektowany tak, aby dekodowanie odbywało się w czasie rzeczywistym nawet na skromnym sprzęcie. FLAC jest kodekiem asymetrycznym, co oznacza, że proces odtwarzania jest bardzo lekki dla procesora, co bezpośrednio przekłada się na niższe zużycie energii i dłuższą pracę na baterii. APE w trybach wysokiej kompresji wykorzystuje złożone algorytmy, które mogłyby przeciążyć procesor bez jednostki FPU.
2. Scenariusz serwerowy: Optymalnym formatem będzie **Monkey’s Audio (APE)**.
Zgodnie z wynikami przedstawionymi w tabeli 1 artykułu, Monkey's Audio oferuje najlepszy współczynnik kompresji (Compression ratio: 1.865) spośród badanych kodeków. W scenariuszu archiwizacji, gdzie priorytetem jest oszczędność miejsca na macierzach dyskowych, wyższy stopień upakowania danych generuje znaczne oszczędności finansowe. Czas kompresji jest pomijalny, ponieważ proces ten odbywa się tylko raz podczas zapisu do archiwum, a rzadki odczyt sprawia, że większe obciążenie procesora przy dekodowaniu nie stanowi problemu systemowego.

**Literatura:** [erk.fe.uni-lj.si/2023/papers/zeleznik(overview_of).pdf](https://erk.fe.uni-lj.si/2023/papers/zeleznik\(overview_of\).pdf)

**Zadanie 1.3: Integralność danych (na podstawie specyfikacji FLAC)**
[FLAC - format](https://web.archive.org/web/20130311161119/https://xiph.org/flac/format.html#stream)
Format FLAC stosuje wielopoziomowy system weryfikacji integralności, co czyni go wyjątkowo odpornym na zjawisko "bit rot":
1. **Poziom ramy (Frame level):** Każda rama posiada w nagłówku **8-bitowy kod CRC**, a cała rama jest zamykana przez **16-bitowy CRC** w stopce (FRAME_FOOTER). Pozwala to na natychmiastowe wykrycie przekłamań bitowych w trakcie odtwarzania.
2. **Poziom strumienia (Stream level):** W bloku metadanych **STREAMINFO** przechowywany jest **128-bitowy skrót MD5** oryginalnego, nieskompresowanego sygnału audio.
3. **Zastosowanie:** Jak podaje specyfikacja: _"Allows the decoder to determine if an error exists in the audio data even when the error does not result in an invalid bitstream"_. Oznacza to, że użytkownik może mieć absolutną pewność co do identyczności kopii z oryginałem, co jest kluczowe w profesjonalnej archiwizacji.