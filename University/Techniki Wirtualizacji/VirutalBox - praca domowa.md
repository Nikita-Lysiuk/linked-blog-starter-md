# Zadanie 1
## Część pierwsza – utworzenie maszyny wirtualnej

![[Pasted image 20251014005016.png]]
Rys. 1: Rozpoczęcie tworzenia maszyny wirtualnej.

![[Pasted image 20251014005046.png]]
Rys. 2: Konfiguracja sprzętu

![[Pasted image 20251014005150.png]]
Rys. 3: Konfiguracja dysku

![[Pasted image 20251014005345.png]]
Rys. 4: Dodatkowa konfiguracja

### Na poprzednich rysunkach (1-4) widać proces tworzenia maszyny wirtualnej bez systemu operacyjnego. 
- Na rysunku 1 podajemy nazwę maszyny wirtualnej i systemu operacyjnego. 
- Na rysunku 2 konfigurujemy sprzęt, wybieramy 2048 MB pamięci RAM i przydzielamy 1 rdzeń procesora. 
- Na rysunku 3 konfigurujemy pojemność dysku wirtualnego na 8 GB. 
- Na rysunku 4 przedstawiono pełną konfigurację maszyny wirtualnej z dodatkowymi parametrami, takimi jak wybór 2 kart sieciowych (nat, hostonly) oraz wybór usb 2.0.
---
## Część druga – utworzenie migawki i instalacja systemu operacyjnego

![[Pasted image 20251014005441.png]]
Rys. 5: Tworzenie migawki bez systemu operacyjnego

![[Pasted image 20251014011556.png]]
Rys. 6: Pomyślna instalacja ubuntu-server i utworzenie odpowiednich migawek

### W drugim etapie tworzę migawkę bez systemu operacyjnego o odpowiedniej nazwie, dodaję plik iso i przechodzę przez proces instalacji systemu operacyjnego, konfigurując nasz serwer (wybór kart sieciowych, utworzenie użytkownika, przypisanie hasła). Po pomyślnej instalacji i konfiguracji tworzę odpowiednią migawkę.
---
## Część 3 – instalacja serwera Apache, a także sprawdzenie statusu serwera za pomocą polecenia sudo systemctl status apache2. Sam serwer został zainstalowany za pomocą polecenia sudo apt install apache2 -y.

![[Pasted image 20251014011906.png]]
![[Pasted image 20251014012147.png]]
Rys. 7: Ostateczny stan naszych migawek

---
---
# Zadanie 2
## Skrypt napisany w bash przy użyciu VBoxManage, z walidacją parametrów wejściowych i automatycznym uruchamianiem po utworzeniu maszyny wirtualnej.

``` bash
#!/bin/bash
# Pliki ISO powinny znajdować się w tym samym folderze co skrypt. 

# --- Sprawdzenie, czy zainstalowany jest VirtualBox CLI (VBoxManage) ---
# Komenda `which VBoxManage` sprawdza, czy narzędzie VBoxManage jest dostępne w systemie.
if ! which VBoxManage > /dev/null 2>&1; then
  echo "VirtualBox nie jest zainstalowany. Zainstaluj go przed uruchomieniem skryptu."
  exit 1
fi

echo "=== Kreator automatycznej instalacji systemu Linux (Debian/Ubuntu) ==="

# --- Wybór systemu operacyjnego ---
echo "Wybierz system do zainstalowania:"
echo "1) Debian 12"
echo "2) Ubuntu Server 24.04 LTS"
read -p "Podaj numer (1 lub 2): " system_choice

# --- Ustawienie nazw ISO w zależności od wyboru ---
# Użytkownik powinien mieć wcześniej pobrane obrazy ISO w tym samym katalogu co skrypt.
if [ "$system_choice" == "1" ]; then
  ISO_FILE="debian-12.0.0-amd64-netinst.iso"
  VM_NAME="Debian12_VM"
  OS_TYPE="Debian_64"
elif [ "$system_choice" == "2" ]; then
  ISO_FILE="ubuntu-24.04-live-server-amd64.iso"
  VM_NAME="UbuntuServer_VM"
  OS_TYPE="Ubuntu_64"
else
  echo "Nieprawidłowy wybór systemu!"
  exit 1
fi

# --- Sprawdzenie, czy plik ISO istnieje ---
if [ ! -f "$ISO_FILE" ]; then
  echo "Brak pliku ISO: $ISO_FILE"
  echo "Upewnij się, że plik ISO znajduje się w tym samym katalogu co skrypt."
  exit 1
fi

# --- Parametry maszyny od użytkownika ---
read -p "Podaj ilość CPU (max 4): " CPU
read -p "Podaj ilość RAM w MB (max 4096): " RAM
read -p "Podaj rozmiar dysku w GB (max 20): " DISK_SIZE
read -p "Podaj typ kontrolera dysku (np. SATA, IDE): " DISK_CONTROLLER
read -p "Podaj typ dysku (np. VDI, VHD, VMDK): " DISK_TYPE
read -p "Podaj ilość kart sieciowych (1 lub 2): " NET_COUNT

# --- Walidacja podstawowa i automatyczna korekcja wartości ---
# Poniższy blok sprawdza, czy użytkownik nie przekroczył maksymalnych wartości.
# Jeśli tak — skrypt ustawia maksymalną dozwoloną wartość i wyświetla komunikat ostrzegawczy zamiast błędu.

# Sprawdzenie CPU
# --- Walidacja parametrów maszyny i korekcja wartości minimalnych/maksymalnych ---

# --- CPU ---
if (( CPU > 4 )); then
  echo "Wybrano zbyt dużą ilość CPU ($CPU). Ustawiono maksymalną wartość: 4."
  CPU=4
elif (( CPU < 1 )); then
  echo "Wybrano zbyt małą ilość CPU ($CPU). Ustawiono minimalną wartość: 1."
  CPU=1
fi

# --- RAM ---
if (( RAM > 4096 )); then
  echo "Wybrano zbyt dużą ilość RAM ($RAM MB). Ustawiono maksymalną wartość: 4096 MB."
  RAM=4096
elif (( RAM < 256 )); then
  echo "Wybrano zbyt mało RAM ($RAM MB). Ustawiono minimalną wartość: 256 MB."
  RAM=256
fi

# --- Dysk ---
if (( DISK_SIZE > 20 )); then
  echo "Wybrano zbyt duży dysk ($DISK_SIZE GB). Ustawiono maksymalną wartość: 20 GB."
  DISK_SIZE=20
elif (( DISK_SIZE < 4 )); then
  echo "Wybrano zbyt mały dysk ($DISK_SIZE GB). Ustawiono minimalną wartość: 4 GB."
  DISK_SIZE=4
fi

# --- Ilość kart sieciowych ---
if (( NET_COUNT > 2 )); then
  echo "Wybrano zbyt dużą ilość kart sieciowych ($NET_COUNT). Ustawiono maksymalną wartość: 2."
  NET_COUNT=2
elif (( NET_COUNT < 1 )); then
  echo "Wybrano zbyt małą ilość kart sieciowych ($NET_COUNT). Ustawiono minimalną wartość: 1."
  NET_COUNT=1
fi

# --- Walidacja kontrolera dysku ---
VALID_CONTROLLERS=("SATA" "IDE" "SCSI")
if [[ ! " ${VALID_CONTROLLERS[@]} " =~ " ${DISK_CONTROLLER} " ]]; then
  echo "Nieprawidłowy kontroler dysku ($DISK_CONTROLLER). Ustawiono domyślnie: SATA."
  DISK_CONTROLLER="SATA"
fi

# --- Walidacja typu dysku ---
VALID_DISK_TYPES=("VDI" "VHD" "VMDK")
if [[ ! " ${VALID_DISK_TYPES[@]} " =~ " ${DISK_TYPE} " ]]; then
  echo "Nieprawidłowy typ dysku ($DISK_TYPE). Ustawiono domyślnie: VDI."
  DISK_TYPE="VDI"
fi

# --- Typ kart sieciowych ---
# Pętla pozwala użytkownikowi wybrać typ każdej karty: NAT lub host-only
declare -a NET_TYPES
for ((i=1; i<=NET_COUNT; i++)); do
  while true; do
    read -p "Wybierz typ karty sieciowej #$i (nat/hostonly): " net_type
    if [[ "$net_type" == "nat" ]] || [[ "$net_type" == "hostonly" ]]; then
      NET_TYPES[i]=$net_type
      break
    else
      echo "Nieprawidłowy typ karty. Wpisz 'nat' lub 'hostonly'."
    fi
  done
done

# --- Dane użytkownika dla instalacji ---
read -p "Podaj nazwę użytkownika: " USERNAME
read -s -p "Podaj hasło: " PASSWORD
echo ""

# --- Tworzenie maszyny wirtualnej ---
echo "Tworzenie nowej maszyny wirtualnej: $VM_NAME ..."
VBoxManage createvm --name "$VM_NAME" --ostype "$OS_TYPE" --register

# --- Konfiguracja sprzętowa VM ---
# Poniższe komendy ustalają podstawowe parametry maszyny: CPU, RAM, EFI, kontroler USB itp.
VBoxManage modifyvm "$VM_NAME" --memory $RAM --cpus $CPU --firmware bios --usb on --usbxhci on

# --- Tworzenie i podłączanie dysku twardego ---
VBoxManage createmedium disk --filename "$VM_NAME.$DISK_TYPE" --size $((DISK_SIZE * 1024)) --format "$DISK_TYPE"
VBoxManage storagectl "$VM_NAME" --name "$DISK_CONTROLLER Controller" --add "$DISK_CONTROLLER" --controller IntelAhci
VBoxManage storageattach "$VM_NAME" --storagectl "$DISK_CONTROLLER Controller" --port 0 --device 0 --type hdd --medium "$VM_NAME.$DISK_TYPE"

# --- Dodanie napędu optycznego i podłączenie ISO ---
VBoxManage storageattach "$VM_NAME" --storagectl "$DISK_CONTROLLER Controller" --port 1 --device 0 --type dvddrive --medium "$ISO_FILE"

# --- Konfiguracja sieci ---
# Ustawienie pierwszej karty sieciowej jako NAT, druga (jeśli istnieje) jako Host-Only.
VBoxManage modifyvm "$VM_NAME" --nic1 nat
if (( NET_COUNT == 2 )); then
  VBoxManage modifyvm "$VM_NAME" --nic2 hostonly --hostonlyadapter2 "vboxnet0"
fi

# --- Instalacja systemu w trybie nienadzorowanym ---
# Komenda `unattended install` instaluje system automatycznie bez interakcji użytkownika.
echo "Rozpoczynanie instalacji nienadzorowanej..."
VBoxManage unattended install "$VM_NAME" \
  --iso="$ISO_FILE" \
  --user="$USERNAME" \
  --password="$PASSWORD" \
  --full-user-name="$USERNAME" \
  --locale="pl_PL" \
  --time-zone="Europe/Warsaw" \
  --install-additions \
  --package-selection-adjustment=minimal \
  --start-vm=headless

# --- Czekanie aż instalacja się zakończy ---
echo "Instalacja w toku... To może potrwać kilka minut."
echo "Po zakończeniu instalacji maszyna uruchomi się automatycznie."

# --- Koniec ---
echo "Instalacja zakończona pomyślnie!"
echo "Maszyna $VM_NAME została uruchomiona."

```