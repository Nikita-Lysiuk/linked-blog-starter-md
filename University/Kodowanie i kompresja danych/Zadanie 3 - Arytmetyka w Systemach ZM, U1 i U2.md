a) 50 
b) 80
c) -30 
d) 100


---
- Dziesiętny $\rightarrow$ ZM $\rightarrow$ U1 $\rightarrow$ U2
- a) 50 $\rightarrow$ 00110010 $\rightarrow$ 00110010 $\rightarrow$ 00110010
- b) 80 $\rightarrow$ 01010000  01010000 $\rightarrow$ 01010000
- c) -30 $\rightarrow$ 10011110 $\rightarrow$ 11100001 $\rightarrow$ 11100010
- d) 100 $\rightarrow$ 01100100 $\rightarrow$ 01100100 $\rightarrow$ 01100100
# Znak-Moduł (ZM)

|    Operandy w ZM         (8 bit)    |                       Opis operacji (słownie)                        |                       Działanie na modułach (7 bit)                        | Wynik ZM           (8 bit) |       Wynik dziesiętny        |                Uzasadnienie                 |
| :---------------------------------: | :------------------------------------------------------------------: | :------------------------------------------------------------------------: | :------------------------: | :---------------------------: | :-----------------------------------------: |
|   50=`00110010` <br>80=`01010000`   |            Dwa dodatnie → dodajemy moduły, znak dodatni.             |   `0110010 + 1010000 = 1 0000010` (carry = 1, wynik modułu = `0000010`)    |             —              | +130 (nie mieści się w 7-bit) |         Moduł 130 > 127 → overflow.         |
|   50=`00110010`<br>–30=`10011110`   | Znaki różne → odejmujemy mniejszy moduł od większego, wynik dodatni. |                       `0110010 – 0011110 = 0010100`                        |         `00010100`         |              +20              | Wynik poprawny, brak przekroczenia zakresu. |
| –30 =`10011110`<br>–100 =`11100100` |              Dwa ujemne → dodajemy moduły, znak ujemny.              | `0011110 + 1100100 =       10000010` (carry = 1, wynik modułu = `0000010`) |             —              | –130 (nie mieści się w 7-bit) |         Moduł 130 > 127 → overflow.         |
|     50=`00110010` 30=`00011110`     |            Dwa dodatnie → dodajemy moduły, znak dodatni.             |                       `0110010 + 0011110 = 1010000`                        |         `01010000`         |              +80              |        Wynik poprawny, bez overflow.        |

# Uzupełnienie do Jedności (U1)
|                     Operandy w U1 (8 bit)                     | Pełne dodawanie (8-bit)                                   | $C_{out}$ | Po korekcji (U1, 8 bit) |      Interpretacja dziesiętna (U1)      |                                                                               Uzasadnienie                                                                               |
| :-----------------------------------------------------------: | :-------------------------------------------------------- | :-------: | :---------------------: | :-------------------------------------: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|             50 = `00110010`  <br> 80 = `01010000`             | `00110010 + 01010000 = 10000010` (9-bit:     `010000010`) |     0     |       `10000010`        | (w U1 → interpretacja bitów = **-125**) | Dwa dodatnie → otrzymano ujemny kod (bit znaku = 1). Wynik matematyczny = +130 nie mieści się w zakresie [-127,+127] → overflow. `10000010` to artefakt overflow (wrap). |
|   50=`00110010` <br> −30 = `11100001` (inwersja `00011110`)   | `00110010 + 11100001 = 00010011` (9-bit: `100010011`)     |     1     | dodajemy 1 → `00010100` |                   +20                   |                                         Cout = 1 → stosujemy end-around carry. Wynik = 20, mieści się w zakresie. Brak overflow.                                         |
| −30 = `11100001` <br> −100 = `10011011` (inwersja `01100100`) | `11100001 + 10011011 = 01111100` (9-bit: `101111100`)     |     1     | po korekcji `01111101`  |    (w U1 → interpretacja = **+125**)    |                Dwa ujemne → wynik powinien być −130 matematycznie (nieprzedstawialne). End-around carry daje `01111101` (= +125) — czyli wrap i overflow.                |
|             50 = `00110010` <br> 30 = `00011110`              | `00110010 + 00011110 = 001010000` (9-bit: `001010000`)    |     0     |       `01010000`        |                   +80                   |                                                   Dwa dodatnie → wynik dodatni i mieści się w zakresie. Brak overflow.                                                   |

# Uzupełnienie do Dwóch (U2) 

|      Działanie      |        Operandy w U2 (8 bit)         |                 Obliczenie binarne | Wynik w U2 (8 bit) | Wynik dziesiętny | Uzasadnienie                                            |     |
| :-----------------: | :----------------------------------: | ---------------------------------: | :----------------: | :--------------: | ------------------------------------------------------- | :-: |
|      $50 + 80$      |     50=`00110010` 80=`01010000`      | +00110010 =01010000   **10000010** |      10000010      |       –126       | Dwa dodatnie → wynik ujemny (bit znaku = 1). (overflow) |     |
|  50 – 30= 50+(–30)  |    50=`00110010`  -30=`11100010`     | +00110010  =11100010  **00010100** |      00010100      |        20        | Wynik poprawny, znak zgodny z oczekiwaniem              |     |
|     –30+(-100)      |    -30=`11100010` -100=`10011100`    | +11100010  =10011100  **01111110** |      01111110      |       +126       | Dwa ujemne → wynik dodatni (overflow)                   |     |
| 50 – (–30) =50 + 30 | 50 = `00110010`  <br>30 = `00011110` | +00110010  =00011110  **01010000** |      01010000      |        80        | Wynik w zakresie, znak poprawny                         |     |
|                     |                                      |                                    |                    |                  |                                                         |     |
- Overflow występuje, gdy dodajemy dwa liczby o tym samym znaku i wynik ma inny znak.

--- 
1) Najprostszy system do implementacji w układzie cyfrowym - **U2 (Two’s Complement)**
	- Dodawanie i odejmowanie realizuje się w **ten sam sposób** — po prostu dodajemy bity.
	- Brak potrzeby sprawdzania znaków operandów ani wykonywania dodatkowych korekcji (jak w U1 — end-around carry) czy analizy znak-moduł (ZM).
	- Overflow można wykryć **prostym XOR** najstarszych bitów operandów i wyniku.
2) System z problemem podwójnego zera - **System: U1**
	- W U1 istnieją **+0 (`00000000`) i −0 (`11111111`)**.
	- Operacje porównywania typu `A = B` stają się trudniejsze, bo trzeba **sprawdzać obie reprezentacje zera** i traktować je jako równe.
	- Dodatkowo może wymagać **dodatkowej logiki w sumatorze lub jednostce porównania**, żeby uniknąć błędnych wyników przy porównaniach lub warunkach sterujących (np. `if A == 0`).