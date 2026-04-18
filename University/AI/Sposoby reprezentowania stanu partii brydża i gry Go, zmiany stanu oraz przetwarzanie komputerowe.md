# Brydż
## Reprezentacja stanów partii
1. **Rozdanie kart (S0)** 
	- **Opis**: Początek gry – 52 karty są rozdzielane na cztery ręce (N, E, S, W), każda po 13 kart.
	- **Reprezentacja**: Zestaw kart dla każdej ręki, np. N: A♠ K♠ Q♠ J♠ 10♠ 9♠ 8♠ 7♠ 6♠ 5♠ 4♠ 3♠ 2♠, S: A♥ K♥...
	- **Przykład**: `[Deal "N:.63.AKQ987.A9732 A8654.KQ5.T.QJT6 J973.J98742.3.K4 KQT2.AT.J6542.85"]`
2. **Licytacja (S1)** 
	- **Opis**: Gracze zgłaszają odzywki, aby ustalić kontrakt (np. 3NT) i rozgrywającego.
	- **Reprezentacja**: Sekwencja odzywek, np. "1♠ - Pas - 2♥ - Pas - 3NT - Pas - Pas - Pas".
	- **Przykład**:  ```[Auction "N"]
				1D      1S   3H =1= 4S
				4NT =2= X    Pass   Pass
				5C      X    5H     X
				Pass    Pass Pass```
3. **Rozgrywka (S2)** 
	- **Opis**: Gracze zagrywają karty w lewach, zbierając punkty za wygrane lewy.
	- **Reprezentacja**: Lista zagranych kart, np. Lewa 1: A♠ (N) - 3♠ (E) - 5♠ (S) - K♠ (W).
	- **Przykład**: 
	 ```[Play "W"]
		SK =1= H3 S4 S3
		C5 C2 C6 CK
		S2 H6 S5 S7
		C8 CA CT C4
		D2 DA DT D3
		D4 DK H5 H7
		- - - H2 ```
 4. **Podliczanie wygranych/przegranych (S3)**
	 - **Opis**: Po rozegraniu 13 lew obliczane są punkty za kontrakt i lewy.
	 - **Reprezentacja**: Tablica punktowa, np. MY: 100 (pod kreską), ONI: 60 (nad kreską).
	 - **Przykład**: `[Score "NS +400"]`.
## Przejścia między stanami
- **S0 → S1**: Rozpoczęcie licytacji – pierwsza odzywka, np. "1C".
- **S1 → S1**: Kolejne odzywki, np. "Pas" lub "2H", aktualizują sekwencję licytacji.
- **S1 → S2**: Trzy "Pas" z rzędu kończą licytację i rozpoczynają rozgrywkę.
- **S2 → S2**: Zagrywanie kart w lewach, np. "AC → 3C → 5C → KC", zmienia stan rąk i lew.
- **S2 → S3**: Po 13 lewach następuje podliczenie punktów, np. "+400" za wygrany kontrakt.
## Portable Bridge Notation (PBN) w przetwarzaniu komputerowym
PBN jest otwartym standardem reprezentowania stanu partii brydża. Reprezentuje on grę w postaci tekstu. 
**Przykład**: 
```1.2  Example game
-----------------
  An example game is described below. The tags are specified in detail
in chapter 3.

[Event "International Amsterdam Airport Schiphol Bridgetournament"]
[Site "Amsterdam, The Netherlands NLD"]
[Date "1995.06.10"]
[Board "1"]
[West "Podgor"]
[North "Westra"]
[East "Kalish"]
[South "Leufkens"]
[Dealer "N"]
[Vulnerable "None"]
[Deal "N:.63.AKQ987.A9732 A8654.KQ5.T.QJT6 J973.J98742.3.K4 KQT2.AT.J6542.85"]
[Scoring "IMP"]
[Declarer "S"]
[Contract "5HX"]
[Result "9"]
{
                S
                H 6 3
                D A K Q 9 8 7
                C A 9 7 3 2
S K Q 10 2                      S A 8 6 5 4
H A 10                          H K Q 5
D J 6 5 4 2                     D 10
C 8 5                           C Q J 10 6
                S J 9 7 3
                H J 9 8 7 4 2
                D 3
                C K 4   
}
[Auction "N"]
1D      1S   3H =1= 4S
4NT =2= X    Pass   Pass
5C      X    5H     X
Pass    Pass Pass
[Note "1:non-forcing 6-9 points, 6-card"]
[Note "2:two colors: clubs and diamonds"]
[Play "W"]
SK =1= H3 S4 S3
C5 C2 C6 CK
S2 H6 S5 S7
C8 CA CT C4
D2 DA DT D3
D4 DK H5 H7
- - -  H2
*
[Note "1:highest of series"] 
```
**Również opis zmiennych**:
``` Upper   Lower   Description     Tags
        -----   -----   -----------     ----
        W       w       West            Dealer, Deal, Declarer, Auction, Play
        N       n       North           Dealer, Deal, Declarer, Auction, Play
        E       e       East            Dealer, Deal, Declarer, Auction, Play
        S       s       South           Dealer, Deal, Declarer, Auction, Play
        S       s       Spades          Deal, Contract, Auction, Play
        H       h       Hearts          Deal, Contract, Auction, Play
        D       d       Diamonds        Deal, Contract, Auction, Play
        C       c       Clubs           Deal, Contract, Auction, Play
        NT      nt      NoTrump         Contract, Auction
        X       x       double          Contract, Auction
        XX      xx      redouble        Contract, Auction
        Pass    pass    Pass            Contract, Auction
        AP      ap      AllPass         Auction
        A       a       Ace             Deal, Play
        K       k       King            Deal, Play
        Q       q       Queen           Deal, Play
        J       j       Jack            Deal, Play
        T       t       Ten             Deal, Play
        None    none    no vuln.        Vulnerable
        Love    love    no vuln.        Vulnerable
        NS      ns      NS vuln.        Vulnerable
        EW      ew      EW vuln.        Vulnerable
        All     all     All vuln.       Vulnerable
        Both    both    All vuln.       Vulnerable
```

![[Pasted image 20250308130926.png]]
# Go
## Stany gry w Go
Gra Go jest rozgrywana na planszy (zazwyczaj 19x19), a jej stan można opisać za pomocą trzech głównych stanów:

1. **Rozpoczęcie gry (S0)**
	- **Opis**: Początkowy stan gry – plansza jest pusta, żaden kamień nie został jeszcze położony. Gracze (Czarny i Biały) przygotowują się do gry.
	- **Reprezentacja**: Macierz 19x19 wypełniona zerami (0 = puste pole).
	- **Przykład**: Macierz: `[[0, 0, ..., 0], ..., [0, 0, ..., 0]]`.
2. **Rozgrywka (S1)**
	- **Opis**: Gracze na przemian kładą kamienie na planszy, próbując otoczyć terytorium lub zbić kamienie przeciwnika. Stan zawiera aktualne pozycje kamieni i historię ruchów.
	- **Reprezentacja**: Macierz 19x19, gdzie 0 = puste, 1 = czarny kamień, 2 = biały kamień, oraz sekwencja ruchów, np. "C3, D4, E5...".
	- **Przykład**: Macierz: `[[0, 0, 1, ...], [0, 2, 0, ...], ...]`, sekwencja: `[C3 (Czarny), D4 (Biały)]`.
3. **Zakończenie gry (S2)**
	 - **Opis**: Gra kończy się, gdy obaj gracze spasują (pass). Podliczane są punkty za terytorium i zbite kamienie.
	 - **Reprezentacja**: Ostateczna macierz planszy i wynik, np. Czarny: 32 punkty, Biały: 28 punktów.
	 - **Przykład**: Macierz: `[[1, 1, 0, ...], [2, 2, 0, ...], ...]`, wynik: `Czarny +4`.
## Przejścia między stanami 
- **S0 → S1**: Pierwszy gracz (Czarny) kładzie kamień, np. na C3, co rozpoczyna rozgrywkę. 
- **S1 → S1**: Kolejne ruchy – gracze kładą kamienie (np. Biały na D4), aktualizując macierz i sekwencję ruchów. W razie potrzeby zbijane są kamienie przeciwnika (usuwane z macierzy). 
- **S1 → S2**: Obaj gracze pasują (dwa "Pass" z rzędu), co kończy rozgrywkę i przechodzi do podliczania punktów.
## Reprezentacja w przetwarzaniu komputerowym 
- **Najlepszy sposób**: Macierz 19x19 (lub mniejsza, np. 9x9). 
- **Dlaczego macierz**: 
	- **Struktura**: Macierz pozwala na szybkie operacje, takie jak sprawdzanie sąsiedztwa, liczenie wolności (otwartych punktów obok grupy kamieni) czy określanie, czy kamień został zbity. 
	- **Efektywność**: Idealna dla algorytmów AI, takich jak Monte Carlo Tree Search (używany w AlphaGo), które wymagają szybkiego dostępu do danych przestrzennych.
	- **Prostota zapisu ruchów**: Ruch to zmiana wartości w macierzy, np. `(3,4) = 1` (Czarny kamień na C3). 
	- **Uniwersalność**: Format macierzy jest łatwy do serializacji (np. do JSON) i przechowywania w plikach. 