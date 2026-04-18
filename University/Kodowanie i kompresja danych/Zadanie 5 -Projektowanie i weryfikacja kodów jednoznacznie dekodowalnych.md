## Zadanie 1.1: Zastosowanie testu na jednoznaczną dekodowalność

**Zasady testu:**
1.  Zaczynamy od $C = S_0$ (zbiór słów kodowych).
2.  W każdym kroku $k \ge 1$ budujemy zbiór "wiszących sufiksów" $S_k$.
3.  **Kod jest non-UD (niejednoznaczny)**, jeśli jakikolwiek wygenerowany zbiór $S_k$ ($k \ge 1$) zawiera słowo kodowe, które już znajduje się w $C$.
4.  **Kod jest UD (jednoznaczny)**, jeśli proces się zatrzyma (wygenerowany zostanie zbiór pusty $\emptyset$) i żaden sufiks z $S_k$ nie pokrył się ze słowem w $C$.

---
## Kod A: {1, 01, 001, 000}

$C = S_0 = \{1, 01, 001, 000\}$
- `1 nie jest prefiksem dla 01, 001, 000`
- `01 nie jest prefiksem dla 1, 001, 000`
- `001 nie jest prefiksem dla 1, 01, 000`
- `000 nie jest prefiksem dla 1, 01, 001`

| Krok | Zbiór sufiksów ($S_k$) | Wyjaśnienie (Porównanie $C$ z $S_{k-1}$) |
| :--- | :--- | :--- |
| $S_1$ | $\emptyset$ (zbiór pusty) | Porównanie $C$ z $C$: Żadne słowo kodowe nie jest prefiksem innego. (To jest kod prefiksowy). |
| $S_2$ | $\emptyset$ | Porównanie $C$ z $S_1 = \emptyset$ zawsze daje $\emptyset$. |

> **Warunek zakończenia:** Proces został zatrzymany, ponieważ wygenerowano zbiór pusty ($S_2 = \emptyset$).
>
> **Wniosek:** Kod A jest **jednoznacznie dekodowalny (UD)**.

---
## Kod B: {00, 11, 010}

$C = S_0 = \{00, 11, 010\}$
- `00 nie jest prefiksem dla 11, 010`
- `11 nie jest prefiksem dla 00, 010`
- `010 nie jest prefiksem dla 00, 11`

| Krok  | Zbiór sufiksów ($S_k$) | Wyjaśnienie (Porównanie $C$ z $S_{k-1}$)                                                      |
| :---- | :--------------------- | :-------------------------------------------------------------------------------------------- |
| $S_1$ | $\emptyset$            | Porównanie $C$ z $C$: Żadne słowo kodowe nie jest prefiksem innego. (To jest kod prefiksowy). |
| $S_2$ | $\emptyset$            | Porównanie $C$ z $S_1 = \emptyset$ daje $\emptyset$.                                          |
|       |                        |                                                                                               |

> **Warunek zakończenia:** Proces został zatrzymany, ponieważ $S_2 = \emptyset$.
>
> **Wniosek:** Kod B jest **jednoznacznie dekodowalny (UD)**.

---
## Kod C: {1, 10, 001, 011}

$C = S_0 = \{1, 10, 001, 011\}$

| Krok  | Zbiór sufiksów ($S_k$) | Wyjaśnienie (Porównanie $C$ z $S_{k-1}$)                                                                                                                                                                                        |
| :---- | :--------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| $S_1$ | **{0}**                | **Porównanie $C$ z $C$:**<br> • Słowo `10` (z $C$) ma prefiks `1` (z $C$). **Sufiks: 0**. <br> Żadne inne słowo nie jest prefiksem dla innych.                                                                                  |
| $S_2$ | **{01, 11}**           | **1. $C$ vs $S_1=\{0\}$**:<br> • `001` (z $C$) ma prefiks `0` (z $S_1$). **Sufiks: 01**.<br> • `011` (z $C$) ma prefiks `0` (z $S_1$). **Sufiks: 11**.                                                                          |
| $S_3$ | **{1}**                | **1. $C$ vs $S_2=\{01, 11\}$**:<br> • `011` (z $C$) ma prefiks `01` (z $S_2$). **Sufiks: 1**.<br> **2. $S_2$ vs $C$**:<br> • `11` (z $S_2$) ma prefiks `1` (z $C$). **Sufiks: 1**.<br> (Oba przypadki dają ten sam sufiks `1`). |

> **Warunek zakończenia:** Proces zatrzymany. Zbiór $S_3 = \{1\}$ zawiera element (`1`), który znajduje się również w $C = S_0$.
>
> **Wniosek:** Kod C **nie jest** jednoznacznie dekodowalny (non-UD).

---
## Zadanie 1.2: Projektowanie iteracyjne (rozbudowa kodu)

### Kandydat 1: 01
Testujemy nowy zbiór: $C_1 = S_0 = \{00, 11, 010, 01\}$

| Krok | Zbiór sufiksów ($S_k$) | Wyjaśnienie (Porównanie $C_1$ z $S_{k-1}$) |
| :--- | :--- | :--- |
| $S_1$ | **{0}** | **$C_1$ vs $C_1$:**<br> • Słowo `010` (z $C_1$) ma prefiks `01` (z $C_1$). **Sufiks: 0**. |
| $S_2$ | **{0, 1, 10}** | **1. $C_1$ vs $S_1=\{0\}$:**<br> • `00` (z $C_1$) ma prefiks `0` (z $S_1$). **Sufiks: 0**.<br> • `010` (z $C_1$) ma prefiks `0` (z $S_1$). **Sufiks: 10**.<br> • `01` (z $C_1$) ma prefiks `0` (z $S_1$). **Sufiks: 1**.<br> **2. $S_1$ vs $C_1$:** Brak. |
| $S_3$ | **{0, 1, 10}** | **1. $C_1$ vs $S_2=\{0, 1, 10\}$:**<br> • `00` / `0` -> **0**; `010` / `0` -> **10**; `01` / `0` -> **1**.<br> • `010` / `01` -> **0**; `11` / `1` -> **1**.<br> **2. $S_2$ vs $C_1$:** Brak.<br> (Zbiór $S_3$ jest identyczny jak $S_2$). |

> **Warunek zakończenia:** Stabilizacja. Osiągnęliśmy stan $S_3 = S_2 = \{0, 1, 10\}$ i żaden element z tego zbioru nie należy do $C_1$ (brak kolizji).
>
> **Werdykt:** Po dodaniu słowa `01`, kod **pozostał** jednoznacznie dekodowalny (UD).

---
### Kandydat 2: 110
Testujemy nowy zbiór: $C_2 = S_0 = \{00, 11, 010, 110\}$

| Krok  | Zbiór sufiksów ($S_k$) | Wyjaśnienie (Porównanie $C_2$ z $S_{k-1}$)                                                                                                                                                  |
| :---- | :--------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| $S_1$ | **{0}**                | **$C_2$ vs $C_2$:**<br> • Słowo `110` (z $C_2$) ma prefiks `11` (z $C_2$). **Sufiks: 0**.                                                                                                   |
| $S_2$ | **{0, 10}**            | **1. $C_2$ vs $S_1=\{0\}$:**<br> • `00` (z $C_2$) ma prefiks `0` (z $S_1$). **Sufiks: 0**.<br> • `010` (z $C_2$) ma prefiks `0` (z $S_1$). **Sufiks: 10**.<br> **2. $S_1$ vs $C_2$:** Brak. |
| $S_3$ | **{0, 10}**            | **1. $C_2$ vs $S_2=\{0, 10\}$:**<br> • `00` / `0` -> **0**; `010` / `0` -> **10**.<br> **2. $S_2$ vs $C_2$:** Brak.<br> (Zbiór $S_3$ jest identyczny jak $S_2$).                            |
|       |                        |                                                                                                                                                                                             |

> **Warunek zakończenia:** Stabilizacja. $S_3 = S_2 = \{0, 10\}$. Żaden element z tego zbioru nie należy do $C_2$ (brak kolizji).
>
> **Werdykt:** Po dodaniu słowa `110`, kod **pozostał** jednoznacznie dekodowalny (UD).

---
### Kandydat 3: 011
Testujemy nowy zbiór: $C_3 = S_0 = \{00, 11, 010, 011\}$

| Krok | Zbiór sufiksów ($S_k$) | Wyjaśnienie (Porównanie $C_3$ z $S_{k-1}$) |
| :--- | :--- | :--- |
| $S_1$ | **$\emptyset$** (zbiór pusty) | **$C_3$ vs $C_3$:**<br>Żadne słowo kodowe nie jest prefiksem innego. Kod $C_3$ jest kodem prefiksowym. |
| $S_2$ | **$\emptyset$** | Porównanie $C_3$ z $S_1 = \emptyset$ zawsze daje $\emptyset$. |

> **Warunek zakończenia:** Wygenerowano zbiór pusty ($S_2 = \emptyset$).
>
> **Werdykt:** Po dodaniu słowa `011`, kod **pozostał** jednoznacznie dekodowalny (UD).

---
Na podstawie przeprowadzonych testów:
* Kandydat 1 (`01`): Zachowuje UD.
* Kandydat 2 (`110`): Zachowuje UD.
* Kandydat 3 (`011`): Zachowuje UD (tworzy kod prefiksowy).

**Odpowiedź:** Wszyscy trzej kandydaci (`01`, `110` oraz `011`) mogą zostać użyci do poprawnego rozszerzenia kodu $B$.

---
## Zadanie 1.3: Oszacowanie efektywności

Na potrzeby tego zadania wybieramy **Kandydata 3** (`011`) z zadania 1.2.

$C = \{00, 11, 010, 011\}$

| Symbol | Słowo kodowe ($c_i$) | Prawdopodobieństwo ($P(s_i)$) | Długość ($d_i$) |
| :--- | :--- | :--- | :--- |
| s1 | `00` | 0.5 | 2 |
| s2 | `11` | 0.2 | 2 |
| s3 | `010` | 0.2 | 3 |
| s4 | `011` | 0.1 | 3 |

---

### 1. Obliczenie średniej długości słowa kodowego ($D$)


$$
D = P(s1) \cdot d1 + P(s2) \cdot d2 + P(s3) \cdot d3 + P(s4) \cdot d4 

$$
$$
D = (0.5 \cdot 2) + (0.2 \cdot 2) + (0.2 \cdot 3) + (0.1 \cdot 3)
$$

> **Wynik:** Średnia długość słowa kodowego dla tego kodu i tych prawdopodobieństw wynosi **2.3 bita/symbol**.

---

### 2. Porównanie z kodem stałorozmiarowym

Do zakodowania 4 różnych symboli ($N=4$) przy użyciu kodu o stałej długości, potrzebujemy $k$ bitów, gdzie $2^k \ge N$.
* $2^2 = 4$ 

Optymalny kod stałorozmiarowy wymagałby **2 bitów na symbol** (np. `00`, `01`, `10`, `11`).

**Porównanie:**
* Nasz kod zmiennorozmiarowy ($D$): 2.3 bita/symbol
* Kod stałorozmiarowy ($D_{st}$): 2.0 bity/symbol

> **Wniosek:** W tym konkretnym przypadku, przy tych prawdopodobieństwach, nasz wybrany kod zmiennorozmiarowy ($D=2.3$) jest **mniej efektywny** niż standardowy kod stałorozmiarowy ($D_{st}=2.0$). Dzieje się tak, ponieważ kod nie został zoptymalizowany (np. metodą Huffmana) – symbol `s1` o wysokim prawdopodobieństwie (0.5) ma tę samą długość (2 bity), co symbol `s2` o znacznie niższym prawdopodobieństwie (0.2).

---

### 3. Odniesienie do standardu ASCII (8 bitów)

* Średnia długość ASCII ($D_{ascii}$): 8.0 bitów/symbol
* Średnia długość naszego kodu ($D$): 2.3 bita/symbol

Aby obliczyć procentową kompresję (redukcję rozmiaru) w stosunku do ASCII, używamy wzoru:

$$
\text{Kompresja} = \left( 1 - \frac{D_{\text{nowy}}}{D_{\text{stary}}} \right) \cdot 100\%
$$
$$
\text{Kompresja} = \left( 1 - \frac{2.3}{8.0} \right) \cdot 100\%
$$
$$
\text{Kompresja} = (1 - 0.2875) \cdot 100\%
$$
$$
\text{Kompresja} = 0.7125 \cdot 100\% = 71.25\%
$$

> **Wniosek:** Przy założeniu, że plik źródłowy składa się tylko z tych 4 symboli (o podanych częstościach), nasz kod oferuje **71.25% kompresji** (oszczędności miejsca) w porównaniu do standardowego kodowania ASCII.