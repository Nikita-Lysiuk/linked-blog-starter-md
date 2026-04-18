# 1.1: Fundament – 7-bitowe ASCII
- Kod: 007! $\rightarrow$ 0x4B 0x6F 0x64 0x3A 0x20 0x30 0x30 0x37 0x21 
# 1.2: Problem polskich znaków 
	„Zażółć gęślą jaźń” 
- Z -> ma kod w 7 bitowym ASCII -> 01011010 (0x5A) -> 01011010 -> `Z`
- a -> ma kod w 7 bitowym ASCII -> 01100001 (0x61) -> 01100001 -> `a`
- `ż` $\rightarrow$ nie ma kodu w 7 bitowym ASCII $\rightarrow$ `10111111` $\rightarrow$ `00111111` $\rightarrow$ `?`
- `ó` $\rightarrow$ nie ma kodu w 7 bitowym ASCII $\rightarrow$ `11110011` $\rightarrow$ `01110011` $\rightarrow$ `s` 
- `ł` $\rightarrow$ nie ma kodu w 7 bitowym ASCII $\rightarrow$ `10110011` $\rightarrow$ `00110011` $\rightarrow$ `3`
- `ć` $\rightarrow$ `11100110` $\rightarrow$ `01100110` $\rightarrow$ `f`
- ` ` $\rightarrow$ spacja jest w ASCII $\rightarrow$ ` `
- `g` $\rightarrow$ `g`
- `ę` $\rightarrow$ `11101010` $\rightarrow$ `01101010` $\rightarrow$ `j`
- `ś` $\rightarrow$ `10011100` $\rightarrow$ `00011100` $\rightarrow$ `FILE SEPARATOR (nie ma symbolu)`
- `l` $\rightarrow$ `l`
- `ą` $\rightarrow$ `10111001` $\rightarrow$ `00111001` $\rightarrow$ `9`
- ` ` $\rightarrow$ ` `
- `j` $\rightarrow$ `j`
- `a` $\rightarrow$ `a
- `ź` $\rightarrow$ `10011111` $\rightarrow$ `00011111` $\rightarrow$ `UNIT SEPARATOR (nie ma symbolu)`
- `ń`$\rightarrow$ `11110001` $\rightarrow$ `01110001` $\rightarrow$ `q`
`Za?s3f gjl9 jaq`
Wszystkie polskie znaki zapisałem w formacie 1250 8 bitów, a następnie skróciłem MSB, przetwarzając tylko 7 bitów, i zakodowałem w ASCII. 
MSB  - czyli **najbardziej znaczący bit**, to bit w liczbie binarnej, który ma **najwyższą wartość pozycyjną** i znajduje się **najbardziej na lewo** w jej zapisie.

# 1.3: Rozwiązanie 1 – strony kodowe (ISO-8859-2)
	Zażółć 
	0x5A 0x61 0xBF 0xF3 0xB3 0xE6 

# 1.4: Rozwiązanie 2 – Unicode i UTF-8
Z - 01011010 
a - 01100001
ż - `017C` $\rightarrow$ `0001 0111 1100` $\rightarrow$ `11000101 10111100` $\rightarrow$ `0xC5BC`
ó - `00F3` $\rightarrow$ `1111 0011` $\rightarrow$ `11000011 10110011` $\rightarrow$ `0xC3B3`
ł - `0142`  $\rightarrow$ `0001 0100 0010` $\rightarrow$ `11000101 10000010` $\rightarrow$ `0xC582`
ć - `0107` $\rightarrow$ `0001 0000 0111` $\rightarrow$ `11000100 10000111` $\rightarrow$ `0xC487`

# 1.5: Problem _Mojibake_ (krzaczki)
1. 
	`50 6F 7A 64 72 6F 77 69 65 6E 69 61 20 7A 20 77 79 6B 6C 61 64 75 21` - "Pozdrowienia z wykladu!" 
2.  `43 7A 65 9C E6 21` 
	To mogło być zapisane w ISO-8859-2, ale 4 bajt został zapisane nieprawidłowo, ponieważ w ISO-8859-2 `ś` zapisuje się jako `0xB6`. Polskie znaki w utf-8 używają 2 bajtów, więc koduje je nieprawidłowo, a dla ASCII brakuje bitu.
3.  Czeć! - coś takiego można zobaczyć po otwarciu pliku w ISO-8859-2
