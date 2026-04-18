Tworzenie nowego profilu o nazwie vm
![[Pasted image 20251117233310.png]]
Ustawienie limitu procesora na 1 oraz ustawienie rozmiaru pamięci RAM na 1GB
![[Pasted image 20251117233258.png]]
Tworzenie nowego magazynu danych. Ponieważ robię pracę na maszynie wirtualnej nie mogę wykorzystać silnik ZFS. Alternatywnie wykorzystuję btrfs. Nadanie magazynu nazwy vm oraz przysnaczenie go na maszyny wirtualne.
![[Pasted image 20251118001521.png]]
Tworzenie nowej bridged network, ustawienie pojemnośći na 40GB oraz nadanie jej nazwy vmnet 
![[Pasted image 20251118001823.png]]
Utworzenie nowego kontenera z systemem debian13, nadanie jej nazwy **wordpress** z wykorzystaniem profilu vm, magazynu danych i sieci vmnet
![[Pasted image 20251118002806.png]]
Za pomocą protokolu ssh odblokowanie możliwości zdalnego połączenia. W podalszych krokach zainstalowanie oprogramowania MariaDB i WordPress![[Pasted image 20251118005236.png]]
Rzut ekranu skonfigurowanego środowiska WordPress w przeglądarce internetowej.
![[Pasted image 20251118013729.png]]