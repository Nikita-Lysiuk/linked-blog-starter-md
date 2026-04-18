# Zadanie 1. Modelowanie zabawki bilardowej jako automat skończony 

### **Definicja**
$$
M = (Q, \Sigma, \delta, q_0, F) 
$$
gdzie:
- $Q$ – zbiór stanów odpowiadających konfiguracjom zastawek $(x_1, x_2, x_3)$.
- $\Sigma = \{A, B\}$  - alfabet wejściowy. 
- $\delta : Q \times \Sigma \to Q$ - funkcja przejścia. 
- $q_0 = (L, L, L)$ - stan początkowy  (wszystkie zastawki skierowane w lewo).
- $F \subseteq Q$ - zbiór stanów akceptujących (gdy kulka wychodzi przez $D$).

### **Zbiór stanów**

Każdy stan to konfiguracja trzech zastawek:
$Q = \{ (L, L, L), (L, L, R), (L, R, L), (L, R, R), (R, L, L), (R, L, R), (R, R, L), (R, R, R) \}$

### **Zbiór stanów akceptujących**

Stan akceptujący to stan, w którym kula opuszcza układ przez $D$. Bez uwzględniania drogi kulki:
$F = \{ (R, L, R), (R, R, R) \}$

### **Funkcja przejścia $\delta$**
$$
\delta ((x_1,x_2,x_3), A) = \begin{cases} 
(\neg x_1, x_2, \neg x3), if \ x_1 = R \\
(\neg x_1, x_2, x3), if \ x_1 = L 
\end{cases}
$$
$$
\delta ((x_1,x_2,x_3), B) = \begin{cases} 
(x_1, \neg x_2, \neg x3), if \ x_2 = L \\
(x_1, \neg x_2, x3), if \ x_2 = R
\end{cases}
$$
## Tablica

| Stany (x1, x2, x3) | A         | B         |
| :----------------: | --------- | --------- |
|    ->(L, L, L)     | (R, L, L) | (L, R, R) |
|     (L, L, R)      | (R, L, R) | (L, R, L) |
|     (L, R, L)      | (R, R, L) | (L, L, L) |
|     (L, R, R)      | (R, R, R) | (L, L, R) |
|     (R, L, L)      | (L, L, R) | (R, R, R) |
|     (R, L, R)*     | (L, L, L) | (R, R, L) |
|     (R, R, L)      | (L, R, R) | (R, L, L) |
|     (R, R, R)*     | (L, R, L) | (R, L, R) |
![[Zad1.excalidraw.svg]]


# Zadanie 2: Uzupełnij poniższy rysunek w ten sposób, by uzyskany 4-stanowy automat nie akceptował żadnego słowa nad alfabetem {a,b} w którym dwie litery b występują obok siebie.
![[zad2.svg]]
# Zadanie 3: Sprawdź czy następujące automaty są równoważne:

![[zad3.excalidraw.svg]]

| Stany    | a                      | b                      |
| -------- | ---------------------- | ---------------------- |
| (q1, q5) | (q2, q6) => (I.S, I.S) | (q1, q5) => (F.S, F.S) |
| (q2, q6) | (q3, q5) => (F.S, F.S) | (q4, q7) => (I.S, I.S) |
| (q3, q5) | (q2, q6) => (I.S, I.S) | (q3, q5) => (F.S, F.S) |
| (q4, q7) | (q4, q7) => (I.S, I.S) | (q4, q7) => (I.S, I.S) |
F.S represents -> Final State.  
I.S represents -> Intermediate State (Non-Final State).
Source: [[[Equivalence Of F.S.A (Finite State Automata) - GeeksforGeeks](https://www.geeksforgeeks.org/equivalence-of-f-s-a-finite-state-automata/)]] 

Automaty są **równoważne**, ponieważ dla każdej pary stanów przejścia i statusy stanów (końcowe lub nie) są zgodne. Nie ma żadnych sprzeczności, więc oba automaty akceptują ten sam język.

# Zadanie 4: Sprawdź, które z poniższych słów należą do języka akceptowanego przez automat:

![[Pasted image 20250316123434.png]]

1. abbbab: (S) 1 -a-> 2 -b-> 3 -b-> 1 -b-> 6 -a-> 6 -b-> 6 (W) ❌
2. aabbaa: (S) 1 -a-> 2 -a-> 5 -b-> 3 -b-> 1 -a-> 2 -a-> 5 (F. S) ✅
3. aaaaaa: (S) 1 -a-> 2 -a-> 5 -a-> 4 -a-> 1 -a-> 2 -a-> 5 (F. S) ✅
4. ababaa: (S) 1 -a-> 2 -b-> 3 -a-> 4 -b-> 4 -a-> 1 -a-> 2 (I. S) ❌
5. ba: (S) 1 -b-> 6 -a-> 6 (W) ❌