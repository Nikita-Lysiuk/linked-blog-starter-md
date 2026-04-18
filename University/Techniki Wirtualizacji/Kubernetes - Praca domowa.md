1. Tworzenia klastera z 2 węzłami. 
![[{E0589D09-1C01-4D72-B7A9-05120036886A}.png]]
2. Plik definiuje nową przestrzeń nazw o nazwie `prod` w celu logicznej izolacji zasobów aplikacji produkcyjnej od reszty klastra.
   ![[{2F74BE29-C3FD-4F5C-B95D-F639D154DC11}.png]]
3. Tworzenie ConfigMap i sekretu dla przestrzeni nazw prod 
   ![[{472BDEAE-9DB2-497F-ADF4-2E9F9F907F2A}.png]]
4. Tworzenie PVC rozmiarem 1Gb dla prod
   ![[{6CCD4902-931C-4049-A04F-EDF5B8896021}.png]]
5. Tworzenie głównego pliku configuracji dla publikacji naszego ngnix serwera
   ![[{86B19A74-1925-479B-9C91-ECF6317A674B}.png]]
6. Usługa typu ClusterIP umożliwiająca komunikację wewnątrz klastra i kierująca ruch do odpowiednich podów aplikacji na porcie 80.
   ![[{8A8C1902-5494-4C5A-AE71-BEC2A0BEA248}.png]]
7. Konfiguracja Ingress definiująca reguły routingu, która kieruje ruch z domeny `author.local` do serwisu aplikacji wewnątrz klastra.
   ![[{DF1CBD00-940D-4079-AF8D-1B038D28AD77}.png]]
8. Konfiguracja Horizontal Pod Autoscaler automatycznie skalująca liczbę podów od 2 do 6 w zależności od obciążenia procesora (próg 50%).![[{228F5F92-B1EE-4FB0-8A92-DA2445276CE3}.png]]
9. Dodawanie hosta do konfiguracji windows
   ![[{DB93A80A-57F0-4E15-9673-25C7F9A2B981}.png]]
   10. Odzyskanie strony 
       ![[Pasted image 20260121200138.png]]