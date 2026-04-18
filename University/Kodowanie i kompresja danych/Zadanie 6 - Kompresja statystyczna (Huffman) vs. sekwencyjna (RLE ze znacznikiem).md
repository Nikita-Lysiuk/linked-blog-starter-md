● **Zbiór A (Dane tekstowe):** BABA_JAGA_BABA
● **Zbiór B (Dane graficzne):** AAAAAAAAAABBBBBBBBBBBBBBBBCCCCCC

## Zadanie 1.1: Kodowanie Huffmana

1. BABA_JAGA_BABA
2. 

|  A  |  6  |
| :-: | :-: |
|  B  |  4  |
|  G  |  1  |
|  J  |  1  |
|  _  |  2  |
|     | 14  |
3.  ![[Pasted image 20251114235035.png]]
4. `0110110010000100011001011011` $\longrightarrow$ $BABA\_JAGA\_BABA$
5. Oryginalny rozmiar: 14 znaków * 8 bit = 112; 
   Skompresowany rozmiar: $2 + 1 + 2 + 1 + 3 + 4 + 1 + 4 + 1 + 3 + 2 + 1 + 2 + 1 = 28$ 
6. **Współczynnik kompresji**: $$\frac{112}{28} = 4$$**Wniosek:** Plik jest **4 razy** mniejszy niż oryginał
   
   **Przyrost względny:** $$1 - \frac{28}{112} = 0.75$$ **Wniosek:** Zaoszczędziliśmy $75\%$ miejsca
   
   **Względny rozmiar ciągu:** $$\frac{28}{112} = 0.25$$ **Wniosek:** Nowy ciąg stanowi $25\%$ rozmiaru oryginału
   
## Zadanie 1.2: Kodowanie RLE ze znacznikiem serii

   **Znacznik serii: $**
   1. **zbiór B** (AAAAAAAAAABBBBBBBBBBBBBBBBCCCCCC).
   2. `$10A$16B$6C`
   3.  
      Oryginalny rozmiar: 32 znaka * 8 bit = 256; 
      Skompresowany rozmiar: $24 + 24 + 24 = 72$ 
   4. **Współczynnik kompresji**: $$\frac{256}{72} \approx 3.56$$**Wniosek:** Plik jest **3.56 razy** mniejszy niż oryginał
   
   **Przyrost względny:** $$1 - \frac{72}{256} \approx 0.718$$ **Wniosek:** Zaoszczędziliśmy $71\%$ miejsca
   
   **Względny rozmiar ciągu:** $$\frac{72}{256} \approx 0.28$$ **Wniosek:** Nowy ciąg stanowi $28\%$ rozmiaru oryginału

   ## Zadanie 1.3: Analiza krzyżowa
5. 
	- `BABA_JAGA_BABA` 
    - Rozmiar zakodowanego ciągu metodą RLE = 112
	- **Wniosek:** ciąg nie został skompresowany, ponieważ nie zawierał powtarzających się znaków. 
6. 
	- `AAAAAAAAAABBBBBBBBBBBBBBBBCCCCCC`
	- 

|  A  | 10  |
| :-: | :-: |
|  B  | 16  |
|  C  |  6  |
![[Drawing 2025-11-12 18.22.18.excalidraw.png]]

   -  `010101010101010101011111111111111111000000000000` $\longrightarrow$ `AAAAAAAAAABBBBBBBBBBBBBBBBCCCCCC`
   - Całkowity rozmiar zakodowanego ciągu: 48 bit


## Zadanie 1.4: Wnioski i porównanie


| Zbiór Danych      | Rozmiar Oryginalny <br>(8 bitów/znak) | Rozmiar po RLE<br>(ze znacznikiem) | Rozmiar po Huffmanie (bity) |
| ----------------- | ------------------------------------- | ---------------------------------- | --------------------------- |
| Zbiór A (Tekst)   | 112                                   | 112                                | 28                          |
| Zbiór B (Grafika) | 256                                   | 72                                 | 48                          |
- Metoda Huffmana dla zbioru A okazała się najlepsza, co nie jest zaskakujące, ponieważ jest to tekst, w którym nie ma powtarzających się ciągów. 
- Dla zbioru B **lepszy był Huffman**, bo zbił rozmiar z 256 bitów do 48 bitów bo Huffman idealnie wykorzystuje **różne częstotliwości symboli** — tutaj `B` dominuje, więc dostaje 1-bitowy kod. RLE nie korzysta z różnicy częstotliwości, tylko patrzy na długość kolejnych powtórzeń.
- RLE ze znacznikiem nie pogarsza pliku, jeśli nie ma serii. W zbiorze A nie było żadnych powtórzeń, więc wszystkie znaki zostały zapisane jako literały, zero ekspansji. Zwykłe RLE (bez znacznika) musiałoby zapisać długość każdej „serii 1”, co prowadziłoby do **rozrostu danych** w 2 razy (licznik + znak).
- Huffman patrzy tylko na **częstotliwości symboli**, a **nie widzi struktury sekwencji**. Dla niego nie ma znaczenia, czy znaki są:
	- rozłożone równomiernie,
	- losowe,
	- czy stoją w jednej gigantycznej serii.
	Dlatego:
	- RLE świetnie kompresuje długie serie,
	- Huffman kompresuje tylko wtedy, gdy rozkład symboli jest nierówny,
	- ale ignoruje informacje o powtórzeniach lokalnych.
	I dlatego, biorąc pod uwagę, że liczba znaków nie jest tak duża (32), może się wydawać, że algorytm Huffmana jest lepszy, a RLE w ogóle nie ma sensu, ale gdyby ciąg znaków składał się ze 100 lub 1000 znaków, co zwykle ma miejsce w przypadku obrazów, widać by było, jak RLE wygrywa bez szans dla Huffmana. 