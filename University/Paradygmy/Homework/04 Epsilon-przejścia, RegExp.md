## Zadanie 10 
- a) ogół sekwencji binarnych, w których występują co najwyżej trzy zera:
	`/^(1*0?1*0?1*0?1*)$/` albo `/^(1*01*01*01*|1*01*01*|1*01*|1*)$/`
- b) ogół sekwencji binarnych, w których występują co najwyżej dwa zera obok siebie:
	`/^(1*|1*(00?1*)*)$/`
- c) ogół sekwencji binarnych, w których liczba zer jest podzielna przez dwa, a liczba jedynek przez **3**: `/^(00)*(111)*$/`
- d)  język poprawnych adresów e-mail. Dla uproszczenia przyjmujemy, że poprawny adres e-mailowy składa się z niepustego identyfikatora złożonego z samych liter (maksymalnie z czterech), znaku "małpy", niepustej nazwy serwera złożonej z samych liter (maksymalnie czterech), kropki oraz dwuliterowej nazwy domeny. Alfabet składa się tylko z liter a, b i c:
	`/^[abc][abc]?[abc]?[abc]?@[abc][abc]?[abc]?[abc]?\.[abc][abc]$/` albo 
	`/^[abc]{1,4}@[abc]{1,4}\.[abc]{2}$/`

## Zadanie 11
![[Drawing 2025-03-22 22.49.08.excalidraw.png]]

## Zadanie 12
Język $L$ ciągów binarnych, w których sekwencja **11** występuje dokładnie raz, nie da się określić wyrażeniem regularnym.
### Dowód teoretyczno-logiczny
Języki regularne nie mają wewnętrznej pamięci i nie potrafią liczyć, ile razy wystąpił dany podciąg. Aby określić, że „11” pojawia się **dokładnie raz**, musielibyśmy śledzić jego liczbę wystąpień, co wymaga pamięci – a więc wykracza poza możliwości automatów skończonych. W związku z tym język $L$ nie jest regularny i nie da się go określić za pomocą wyrażenia regularnego.

### Dowód metodą pompowania
Załóżmy, że język $L$ jest regularny. Wtedy spełnia lemat o pompowaniu: istnieje stała $p$, dla której każdy ciąg $w$ o długości $∣w∣≥p$ można podzielić na $w=xyz$, gdzie:
1. $xy^{k}z \in L$ dla dowolnego $k \geq 0$,
2. $|xy| \leq p$ ,
3. $|y| > 0$.
Wybierzmy $w=101101$, który należy do $L$.
Niech $x=10, y=1, z=101$ 
Dla $k=2$ otrzymujemy $xy^{2}z=1011101$, który zawiera dwa wystąpienia $\textquotedblleft 11\textquotedblright$, więc $xy^{k}z \notin L$ 