## Zadanie 1.1: Magiczne liczby (identyfikacja plików)

Na podstawie danych [z tabeli z Wikipedii](https://en.wikipedia.org/wiki/List_of_file_signatures) można dowiedzieć się, że sygnatury plików dla png, gif, bpm, jpg są następujące:
- PNG - `89 50 4E 47 0D 0A 1A 0A` ‰PNG␍␊␚␊ (ISO 8859-1)
- GIF - `47 49 46 38 37 61` GIF87a (ISO 8859-1) albo `47 49 46 38 39 61` dla GIF89a (ISO 8859-1)
- BMP - `42 4D` BM (ISO 8859-1)
- JPG - `FF D8 FF E0 00 10 4A 46 49 46 00 01` ÿØÿà␀␐JFIF␀␁ (ISO 8859-1)
Jak widać na zrzutach ekranu, każdy format ma swoją odpowiednią sygnaturę. 
PNG: ![[Pasted image 20251130161150.png]]
GIF: ![[Pasted image 20251130161227.png]]
BMP: ![[Pasted image 20251130161317.png]]
JPG: ![[Pasted image 20251130161424.png]]
## Zadanie 1.3: Ręczna ekstrakcja danych
1. BMP:
	- Szerokość obrazu: `80 02 00 00` Jest to zapisane w porządku little endian, czyli w Big Endian byłoby to` 0x280` co odpowiada 640d.
	- Wysokość obrazu: `AA 01 00 00` -> `0x1AA` (Big endian) = 426d.
	- Głębię bitów `18 00` 24 bity.
2. PNG:
	- ![[Pasted image 20251130163845.png]]
	- Szerokość obrazu: `00 00 13 88` czyli `0x1388` -> 5000d.
	- Wysokość obrazu: `00 00 13 88` -> 5000d. (bo mam obraz 5000x5000)

## Zadanie 1.4: Inspekcja metadanych EXIF
- Model aparatu: Canon PowerShot G15
- Wartość ISO: 800
- Data wykonania zdjęcia: 2024:12:31 20:09:31+0000
- Ogniskowa: 9.8 mm
- FOV: 43.2 deg
- Największym zagrożeniem jest **ujawnienie dokładnej geolokalizacji (danych GPS)**, gdzie zdjęcie zostało wykonane. Publiczne udostępnianie takich zdjęć może prowadzić do określenia Twojego miejsca zamieszkania, a tym samym stwarza **ryzyko stalkingu** lub naruszenia bezpieczeństwa osobistego.