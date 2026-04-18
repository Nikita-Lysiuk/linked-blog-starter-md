Tworzenie kontenera z obrazu debian:12. Pobiera się go z docker hub. 
![[Pasted image 20251102192815.png]]
To polecenie uruchamia wcześniej zatrzymany kontener `my_debian` i od razu **podłącza** (`-a`) do niego terminal w trybie **interaktywnym** (`-i`), pozwalając wprowadzać polecenia.
![[Pasted image 20251102192824.png]]
Instalacja htop
![[Pasted image 20251102193124.png]]
Створення образу на основі контейнера 
![[Pasted image 20251102194033.png]]
Historia warstw w nowym wydaniu
![[Pasted image 20251102194543.png]]
Tworzenie kontenera na podstawie nowego obrazu htop z ograniczoną liczbą rdzeni i pamięci operacyjnej po 1 oraz montowaniem woluminów 
![[Pasted image 20251102200300.png]]
Utworzenie pliku w katalogu volumes /dane w systemie gospodarza i wpisanie do niego imienia i ulubionego koloru. Sprawdzenie, czy plik ten jest widoczny w kontenerze. 
![[Pasted image 20251102201047.png]]
![[Pasted image 20251102201240.png]]
Przypisanie wyniku top do pliku w tej samym folderze /dane
![[Pasted image 20251102202544.png]]
![[Pasted image 20251102202829.png]]
Usuwanie kontenerów 
![[Pasted image 20251102203330.png]]
Pobieranie oficjalnego obrazu linuxserver/openssh-server, tworzenie kontenera na jego podstawie z parametrami konfiguracyjnymi. Jednym z parametrów jest ustawienie publicznego klucza Rsa do łączenia się za pomocą kluczy, a nie haseł.
![[Pasted image 20251102211138.png]]
Połączenie z kontenerem przez ssh przez port zewnętrzny i prywatny klucz rsa
![[Pasted image 20251102211201.png]]
