## ==Zadanie 1.1: Reprezentacja zmiennych w pamięci (Endianness)==

Mając tę liczbę `305419896` $\longrightarrow$ `0x12345678` i adres początkowy `0x00FF0000`
można zapisać tę tabelę w następujący sposób. W przypadku little endian kolejność będzie zasadniczo od końca do początku, a w przypadku big endian odwrotnie.

| Adres pamięci | Wartość (HEX) (Little-Endian) | Wartość (HEX) (Big-Endian) |
| ------------: | ----------------------------: | -------------------------: |
|    0x00FF0000 |                          0x78 |                       0x12 |
|    0x00FF0001 |                          0x56 |                       0x34 |
|    0x00FF0002 |                          0x34 |                       0x56 |
|    0x00FF0003 |                          0x12 |                       0x78 |

## ==Zadanie 1.2: Wyrównanie pamięci (_Padding_)==
``` C
struct Student {
   char inicjal; // 1 bajt
   int numer_id; // 4 bajty
   short wiek; // 2 bajty
}
```
1. Teoretycznie, jeśli wziąć pod uwagę tę strukturę, jej rozmiar powinien wynosić 7 bajtów.
2. Ale w rzeczywistości zajmuje aż 12 bajtów, czyli o 41,6% więcej niż teoretycznie powinno.
3. Zamiast rysunku postanowiłem w praktyce napisać kod, spróbować go debugować i zobaczyć, jak naprawdę działa komputer. Do debugowania zdecydowałem się użyć debuggera **gdb**, ponieważ było to najprostsze rozwiązanie, biorąc pod uwagę brak konfiguracji **C++** w **vscode**, ale jest on dość potężny i zapewnia wszystko, czego potrzeba. Jak widać, rozmiar zmiennej typu **Student** wynosi w rzeczywistości **12 bajtów**. Na mapie pamięci (u mnie wyświetla się **Little Endian**) widać, że padding jest w zasadzie śmieciem. Pierwsze **4 bajty** to **char**, gdzie `0xbaadf04d`, gdzie w zasadzie `0x4d` jest naszym **char** i jest to `'M'` w **ASCII**. Dalej jest zwykły **int**, gdzie wszystkie **4 bajty** są zmienną, a w **hex** to `0x0004e53c`, co w systemie dziesiętnym wynosi   **320828**, a na końcu mamy **short**, który zajmuje **2 bajty**, a więc `0xbaad0013`, gdzie `0x0013` to w systemie dziesiętnym **19**.
![[Pasted image 20251203232815.png]]
## Zadanie 1.3: Segmentacja pamięci procesu

| **Elementy programu**                                          | **Segmenty pamięci**                                         |
| :------------------------------------------------------------- | :----------------------------------------------------------- |
| Zmienna lokalna int x = 5; <br>wewnątrz funkcji main.          | **Stos (Stack)** – zmienne lokalne, parametry funkcji        |
| Kod maszynowy funkcji <br>(instrukcje procesora).              | **Segment Kodu (Text Segment)** – tylko do odczytu           |
| Zmienna dynamicznie zaalokowana <br>operatorem new lub malloc. | **Sterta (Heap)** – pamięć dynamiczna                        |
| Zmienna globalna int g_licznik=0;.                             | **Segment Danych (Data/BSS)** – zmienne statyczne i globalne |
| Stała tekstowa (string literal) <br>"Witaj świecie".           | **Segment Kodu (Text Segment)** – tylko do odczytu           |
|                                                                |                                                              |
## Zadanie 1.4: Arytmetyka wskaźników
ptr = `0x1000`
1) `ptr + 1  = 0x1004`
2) `(char*)ptr + 1 = 0x1001`
3) `ptr + 4 = 0x1010`