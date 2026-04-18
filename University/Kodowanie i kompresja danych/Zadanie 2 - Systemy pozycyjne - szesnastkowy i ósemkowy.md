**System pozycyjny** - metoda zapisywania liczb, w której używa się tylko określonych cyfr, a każdą liczbę można zapisać w postaci ciągu tych cyfr, z których każda jest mnożną potęgi liczby stanowiącej podstawę danego systemu (podstawa b)

**Podstawa b** - liczba całkowita $b \geq 2$ która określa liczbę dopuszczalnych cyfr w systemie i wagę pozycji.
## **Opis systemów**
1) **System szesnastkowy**
	- **Podstawa**: $b = 16$.
	- **Ciąg znaków**: $\{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E, F\}$,  gdzie A = 10, B = 11, C = 12, D = 13, E = 14, F = 15.
	- **Zapis**: Zazwyczaj prefiks `0x` lub sufiks `h` (na przykład `0x2A`, `2Ah`).
	- **Zastosowanie w informatyce**: Jest powszechnie używany w informatyce, ponieważ pozwala na łatwiejsze i bardziej zwarte przedstawienie danych binarnych (np. jeden bajt można zapisać jako dwa znaki heksadecymalne (szestnastkowe)).
	- **Przykłady zastosowań**: 
		- **Kolory w grafice**: W trybie RGB, kolor jest reprezentowany przez trzy pary cyfr szesnastkowych (np. `#FFFFFF` dla bieli).
		- **Adresy MAC**: Służy do adresowania urządzeń w sieciach komputerowych, np. 28-80-23-D6-BE-14.

2) **System ósemkowy**
	- **Podstawa**: $b = 8$
	- **Ciąg znaków**: $\{0, 1, 2, 3, 4, 5, 6, 7\}$.
	- **Zapis**: Zazwyczaj z prefiksem `0o` lub w niektórych językach programowania z wiodącym zerem (np. `0755`).
    - **Zastosowanie w informatyce**: System ósemkowy jest używany głównie tam, gdzie dane binarne można wygodnie grupować w trójki bitów — każda cyfra ósemkowa odpowiada dokładnie **3 bitom**.
    - **Przykłady zastosowań**:
	    - **Uprawnienia w systemach UNIX/Linux**: Prawa dostępu do plików i katalogów są często reprezentowane w zapisie ósemkowym (np. `0755` oznacza `rwxr-xr-x`).
        - **Stare systemy komputerowe i mikrokontrolery**: Był powszechny w starszych architekturach komputerowych, gdzie dane były przechowywane w grupach po 3 bity.
        - **Reprezentacja danych binarnych**: Stosowany do skróconego zapisu wartości binarnych w kontekście technicznym, szczególnie w urządzeniach wbudowanych.
## **Algorytmy konwersji**
1) **Algorytmy konwersji: szesnastkowy $\rightarrow$ ósemkowy**: Aby przekonwertować system szesnastkowy na system ósemkowy, najbezpieczniej jest użyć warstwy pośredniej, którą może być system dziesiętny lub binarny. W przypadku tego algorytmu zostanie użyty system binarny. 
	1) **Algorytm**:
		1) Dla każdej cyfry `hex` należy znaleźć jej 4-bitowy zapis binarny. 
		2) Należy połączyć wszystkie czterobitowe bloki w jeden ciąg bitów, w tej samej kolejności.
		3) Należy pogrupować bity po trzy, licząc od prawej strony (LSB). Jeśli na lewym końcu pozostanie 1 lub 2 bity — dodać po lewej stronie brakujące zera, tak aby każda grupa miała 3 bity.
		4) Należy zastąpić każdą trójkę bitów cyfrą ósemkową. `000→0, 001→1, 010→2, 011→3, 100→4, 101→5, 110→6, 111→7`
		5) Na koniec złóżyć cyfry od lewej do prawej — to jest wynik.
	2) **Przykład obliczenia**: Weźmy: **`0x2A3F`**
		1) Rozbijamy na poszczególne cyfry `hex` 
		2) Zastępujemy 4-bitowymi blokami 
			- `2` $\rightarrow$ `0010`
			- `A` $\rightarrow$ `1010`
			- `3` $\rightarrow$ `0011`
			- `F` $\rightarrow$ `1111`
			`0010 1010 0011 1111` $\rightarrow$ `0010101000111111` - ciąg bitowy 
		3) Grupowanie w trójki od prawej:
			- Długość= 16; $16\mod3 = 1 \rightarrow$ brakujące 2 bity z lewej.
			- Dopisujemy `00` z lewej: $\rightarrow$ pełny ciąg `000010101000111111`
			- Teraj grupujemy (od lewej): `000 010 101 000 111 111`.
		4) Zamiana trójek na cyfry ósemkowe:
			- `000` → `0`
			- `010` → `2`
		    - `101` → `5`
		    - `000` → `0`
		    - `111` → `7`
		    - `111` → `7`
			Zatem ciąg ósemkowy: `0 2 5 0 7 7` → po usunięciu wiodącego zera: **`25077`**.
		5) Szybka weryfikacja:
			- `0x2A3F` w dziesiętnym = $2 \cdot 16^{3} + 10 \cdot 16^{2} + 3\cdot16^{1} + 15\cdot16^{0} = 8192 + 2560 + 48 + 15 = 10815$.
		    - `0o25077` w dziesiętnym = $2\cdot8^{4}+5\cdot8^{3}+0\cdot8^{2}+7\cdot8^{1}+7\cdot8^{0} = 8192 + 2560 + 0 + 56 + 7 = 10815$
2) **Algorytmy konwersji: ósemkowy $\rightarrow$ dziesiętny**:
	1) **Algorytm**:
		1) Zapisać liczbę ósemkową w postaci ciągu cyfr (bez prefiksu). Przykład: `1753`.
		2) Dla każdej cyfry $d_{k}$​ obliczyć udział: $u_k = d_k \cdot 8^k$.
			- Najpierw policzyć kolejne potęgi 8: $8^0 = 1, 8^1 = 8, 8^2 = 64,...$
			- Następnie pomnóżyć każdą potęgę przez odpowiednią cyfrę $d_k$.
		3) Zsumować wszystkie udziału: wynik = $\sum_ku_k$.
		4) Wynik to liczba w systemie dziesiętnym.
	2) **Przykład obliczenia**: Weźmy: **`1753`** 
		1) Rozkład cyfr (od lewej): `1 7 5 3`. Pozycje (od prawej):
			- $d_0 = 3 \cdot (8^0) = 3 \cdot 1$
		    - $d_1 = 5 \cdot (8^1) = 5 \cdot 8$
		    - $d_2 = 7 \cdot (8^2) = 7 \cdot 64$
		    - $d_3 = 1 \cdot (8^3) = 1 \cdot 512$
		2) Udziały $u_k = d_k * 8^k$:
			- $u_0 = 3 \cdot 1 = 3$
			- $u_1 = 5 \cdot 8 = 40$
			- $u_2 = 7 \cdot 64 = 448$
			- $u_3=1 \cdot 512 = 512$
		3) Suma:
			- $3 + 40 = 43$
			- $43 + 448 = 491$
			- $491 + 512 = 1003$
		Wynik: $1743_8 = 1003_{10}$
		