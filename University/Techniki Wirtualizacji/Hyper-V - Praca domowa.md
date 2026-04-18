Kreator tworzenia maszyny wirtualnej w menedżerze Hyper-V i konfiguracja maszyny wirtualnej 
Konfiguracja nazwe maszyny
![[Pasted image 20251019144706.png]]
Wybór 1 generacji 
![[Pasted image 20251019144717.png]]
Przydzielenie 1024 MB pamięci RAM i ustawienie typu dynamicznego, aby system sam wykorzystywał ilość pamięci, której potrzebuje. 
![[Pasted image 20251019144744.png]]
Virtualny dysk twardy na 8GB
![[Pasted image 20251019144850.png]]
Montowanie ISO z ubuntu serwer 
![[Pasted image 20251019144921.png]]
Konfiguracja pamięci dynamicznej RAM wskazanie zakresu dopuszczalnego wykorzystania pamięci 
![[Pasted image 20251019145011.png]]
Tworzenie i konfiguracja 2 adapterów internetowych, jednego do połączenia wewnętrznego, a drugiego do połączenia zewnętrznego
![[Pasted image 20251019145725.png]]
![[Pasted image 20251019184111.png]]
Po zainstalowaniu i skonfigurowaniu systemu operacyjnego utworzyłem migawkę  z czystym systemem operacyjnym.
![[Pasted image 20251019184200.png]]
Instalacja OpenVPN za pomocą polecenia: `sudo apt install openvpn easy-rsa -y` 
![[Pasted image 20251019184630.png]]
Pomyślnie skonfigurowano OpenVPN
![[Pasted image 20251021235131.png]]
Wykorzystanie pamięci RAM z openVPN
![[Pasted image 20251021235246.png]]
Instalcja apache serwera za pomocą `sudo apt install apache2` i liczba pamięci potrzebna na go działanie 
![[Pasted image 20251021235534.png]]
Ilość pamięci przy niezainstalowanych programach. Po analizie widać, że OpenVPN sam w sobie nie wymaga pamięci RAM i że bez niego ilość pamięci RAM nie uległa zmianie (do momentu użycia samego OpenVPN), apache2 sam w sobie hostuje serwer, więc ponieważ jest aktywnie używany, zaczyna wykorzystywać pamięć RAM. W obu przypadkach ilość dostępnej pamięci wzrosła po zainstalowaniu programów, ponieważ maszyna wirtualna liczy na większe wykorzystanie pamięci RAM.
![[Pasted image 20251021235730.png]]
3 migawki: Pusty system, system tylko z openVPN oraz system tylko z apache2
![[Pasted image 20251021235746.png]]
Tworzenie kopii na podstawie wyeksportowanej maszyny wirtualnej
![[Pasted image 20251022000102.png]]
![[Pasted image 20251022000519.png]]![[Pasted image 20251022000834.png]]
![[Pasted image 20251022000845.png]]
Po utworzeniu 2 kopii i przywróceniu migawek mamy dwie niezależne maszyny wirtualne tylko z oprogramowaniem odpowiadającym nazwie maszyny wirtualnej. 