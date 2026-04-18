**Modus Ponens** - $((p\Rightarrow q)\wedge p) \Rightarrow q \Leftrightarrow [\neg((\neg p \vee q) \wedge p) \vee q]$
7. $\forall{x} \forall{y} \ \ Człowiek(x) \land Władca(y) \land pdz(x, y) \Rightarrow \neg Loyalny(x,y)$
	CNF: $\forall{x} \forall{y} \ \ \neg (Człowiek(x) \land Władca(y) \land pdz(x, y)) \lor \neg Loyalny(x,y)$
	Після права де Моргана
	$\forall{x} \forall{y} \ \ \neg Człowiek(x) \lor \neg Władca(y) \lor \neg Pdz(X, y) \lor \neg Lojalny(x, y)$
	
M - Markus
C - Cesar

$\neg Człowiek(M) \lor \neg Władca(C) \lor \neg Pdz(M, C) \lor \neg Lojalny(M, C)$
~~(Człowiek(M)~~ $\Rightarrow (\neg Władca(C) \lor \neg Pdz(M, C) \lor \neg Lojalny(M, C)) \land$ ~~Człowiek(M)~~
Dokonujemy rezolucji (7) z (1)
A) $\neg Władca(C) \lor \neg Pdz(M, C) \lor \neg Lojalny(M, C)$

$(Władca(C) \Rightarrow (\neg Pdz(M, C) \lor \neg Loyalny(M, C)) \land Władca(C)$
Dokonujemy rezolucji (A) z (4)
B) $\neg Pdz(M, C) \lor \neg Loyalny(M, C)$

$(Pdz(M, C) \Rightarrow \neg Loyalny(M, C)) \land Pdz(M, C)$
Dokonujemy rezolucji (B) z (8)
C) $\neg Lojalny(M, C)$

9) Lojalny(M, C) ?
Dokonujemy rezolucji (C) z (9)
$\neg Lojalny(M,C) \land Lojalny(M, C)$
анти-Тавтологія  

---
 # Zadanie 2
- Stałe indywidułowe:
	 - J - Jan; 
	 - A - Adam; 
	 - B - Basia;
- Predykaty:
	- Lubi(x, y);
	- Pożywienia(x);
	- Jabłko(x);
	- Kurczak(x);
	- Zabija(x, y);
	- Orzeszki(x);
	- Je(x, y);
	- Żyje(x);
- Clauzule
	1. $\forall{x} \ \ Pożywienie(x) \Rightarrow Lubi(Jan, x)$
	2. $\forall{x} \ \ Jabłko(x) \Rightarrow Pożywienie(x)$
	3. $\forall{x} \ \ Kurczak(x) \Rightarrow Pożywienie(x)$
	4. $\forall{x} \forall{y} \ \ Je(x, y) \land \neg Zabija(x, y) \Rightarrow Pożywienia(x)$
	5. a) $\forall{x} \ \ Orzeszki(x) \Rightarrow Je(A, x)$
	6. b) $Żyje(A)$
	7. $\forall{x} \ \ Je(A, x) \Rightarrow Je(B, x)$


CPN: 
1. $\forall{x} \ \ \neg Pożywienie(x) \lor Lubi(Jan,x)$
2. $\forall{x} \ \ \neg Jabłko(x) \lor Pożywienie(x)$
3. $\forall{x} \ \ \neg Kurczak(x) \lor Pożywienie(x)$
4. $\forall{x}\forall{y} \ \ \neg Je(x, y) \lor Zabija(x, y) \lor Pożywienie(x)$
5. a) $\forall{x}  \ \ \neg Orzeszki(x) \lor Je(A, x)$
6. b) $Żyje(A)$
7. $\forall{x} \ \ \neg Je(A, x) \lor Je(B, x)$
---
## Teza: $\forall{x} \ \ Orzeszki(x) \Rightarrow Lubi(Jan, x)$
$\forall{x} \ \ \neg Orzeszki(x) \lor Lubi(Jan, x)$
## Antyteza: $\forall{x} \ \ \neg(\neg Orzeszki(x) \lor Lubi(Jan, x))$
$\exists{x} \ \ Orzeszki(x) \land \neg Lubi(jan, x)$

- Antyteza(A): $\exists{x} \ \ Orzeszki(x)$
- Antyteza(B): $\exists{x} \ \ \neg Lubi(Jan, x)$

Добавляємо нову сталу $C$ яка позначає якийсь один горіх, якого Ян не любить
$\forall{x} \ \ \neg Pożywienie(x) \lor Lubi(Jan, x)$
1. $\neg Pożywienie(C) \lor Lubi(Jan, C)$
	$(\neg Lubi(Jan, C) \Rightarrow \neg Pożywienie(C))$
	Dokonujemy rezolucji z Antyteza(B)
	$((\neg Lubi(Jan, C) \Rightarrow \neg Pożywienie(c)) \land \neg Lubi(Jan, C) \Rightarrow \neg Pożywienie(C))$
	$I$: $\neg Pożywienie(C)$
	Dokonujemy rezolucji 4 z $I$ 
	$II$: $\neg Je(H, C) \lor Zabija(C, H)$
	$\exists{x} \ \ Zabija(x, H) \Leftrightarrow \neg Żyje(H)$
	5b) $Zyje(Adam) \Leftrightarrow \neg Zabija(x, Adam)$
	5b) $\neg Zabija(C, Adam)$
	Robimy rezolucji 5b z $II$
	$III$: $\neg Je(H, C)$
	5a) $\neg Orzeszki(C) \lor Je(A, C)$
	Dokonujemy rezolucji 5a z $III$
	$IV$: $\neg Orzeszki(C)$
	$IV \land Antyteza(A)$
	Вийшла тавтологія, отже теза правдива

 
	
	











