## Stany: 
## Stany
Każdy stan można opisać jako `(ML, KL, MR, KR, Ł)`, gdzie:
- `ML` – liczba misjonarzy na lewym brzegu,
- `KL` – liczba kanibali na lewym brzegu,
- `MR` – liczba misjonarzy na prawym brzegu,
- `KR` – liczba kanibali na prawym brzegu,
- `Ł` – położenie łodzi (`L` dla lewego brzegu, `R` dla prawego brzegu).

### Stan początkowy: 
`(3, 3, 0, 0, L)`

### Stan końcowy:
`(0, 0, 3, 3, R)`

## Możliwe akcje
## Możliwe akcje

1) **1 kanibal L → R**  
   **Akcja** `(ML, KL, MR, KR, L | KL > 0, KR+1 <= MR ∨ MR = 0) => (ML, KL-1, MR, KR+1, R)`
2) **2 kanibali L → R**  
   **Akcja** `(ML, KL, MR, KR, L | KL > 1, KR+2 <= MR ∨ MR = 0) => (ML, KL-2, MR, KR+2, R)`
3) **1 misjonarz L → R**  
   **Akcja** `(ML, KL, MR, KR, L | ML > 0, (ML-1 >= KL ∨ ML-1 = 0), KR <= MR+1) => (ML-1, KL, MR+1, KR, R)`
4) **2 misjonarzy L → R**  
   **Akcja** `(ML, KL, MR, KR, L | ML > 1, (ML-2 >= KL ∨ ML-2 = 0), KR <= MR+2) => (ML-2, KL, MR+2, KR, R)`
5) **1 misjonarz i 1 kanibal L → R**  
   **Akcja** `(ML, KL, MR, KR, L | ML > 0, KL > 0, (ML-1 >= KL-1 ∨ ML-1 = 0), KR+1 <= MR+1) => (ML-1, KL-1, MR+1, KR+1, R)`
6) **1 kanibal R → L**  
   **Akcja** `(ML, KL, MR, KR, R | KR > 0, KL+1 <= ML ∨ ML = 0) => (ML, KL+1, MR, KR-1, L)`
7) **2 kanibali R → L**  
   **Akcja** `(ML, KL, MR, KR, R | KR > 1, KL+2 <= ML ∨ ML = 0) => (ML, KL+2, MR, KR-2, L)`
8) **1 misjonarz R → L**  
   **Akcja** `(ML, KL, MR, KR, R | MR > 0, ML+1 >= KL) => (ML+1, KL, MR-1, KR, L)`
9) **2 misjonarzy R → L**  
   **Akcja** `(ML, KL, MR, KR, R | MR > 1, ML+2 >= KL) => (ML+2, KL, MR-2, KR, L)`
10) **1 misjonarz i 1 kanibal R → L**  
   **Akcja** `(ML, KL, MR, KR, R | MR > 0, KR > 0, ML+1 >= KL+1) => (ML+1, KL+1, MR-1, KR-1, L)`


![[Drawing 2025-03-21 20.50.34.excalidraw.svg]]
