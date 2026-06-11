# Rozwiązanie egzaminu INF.02-01-26.05-SG

**Wersja arkusza:** SG
**Sesja:** Czerwiec 2025 (Zgodnie z poleceniem)

---

## 1. Wykonaj montaż okablowania sieciowego

**Zgodnie z wytycznymi w `README.md` (Punkt A):** Pominąłem zadanie dotyczące montażu okablowania sieciowego.
*(Zadanie polegało na zarobieniu kabla U/UTP z obu stron wtykiem 8P8C według standardu T568B).*

---
## 2. Skonfiguruj ruter (VLAN-y i routing)
Zgodnie z wytyczną `B` (`README.md`), konfigurację oparto o urządzenia **MikroTik**.

**Wymagania:**
*   Wyłączony DHCP.
*   Trzy sieci VLAN (802.1Q) przypisane do portu 2 z tagowaniem (trunk):
    * ID = 1, IP: `10.10.1.1/24`, Nazwa np.: `VLAN1`
    * ID = 2, IP: `10.10.2.1/25`, Nazwa np.: `VLAN2`
    * ID = 3, IP: `10.10.3.1/26`, Nazwa np.: `VLAN3`
*   Włączony routing między sieciami VLAN.

**Rozwiązanie w MikroTik (WinBox / CLI Terminal):**
```routeros
# Jeżeli istnieje wyczyść konfig domyślny
/system reset-configuration no-defaults=yes skip-backup=yes

# Wyłączenie ew. domyślnego DHCP
/ip dhcp-server remove [find]

# Utworzenie interfejsów VLAN przypisanych do fizycznego portu np. ether2 (tryb trunk):
/interface vlan add name=VLAN1 vlan-id=1 interface=ether2
/interface vlan add name=VLAN2 vlan-id=2 interface=ether2
/interface vlan add name=VLAN3 vlan-id=3 interface=ether2

# Przypisanie adresacji IP do nowoutworzonych interfejsów VLAN
/ip address add address=10.10.1.1/24 interface=VLAN1
/ip address add address=10.10.2.1/25 interface=VLAN2
/ip address add address=10.10.3.1/26 interface=VLAN3
```
*Uwaga: W systemie RouterOS, jeżeli przypiszesz adresy IP do różnych interfejsów (w tym VLAN), router automatycznie dodaje wpisy typu "Dynamic Active Connected" (DAC) do tablicy routingu (włączając routing między nimi).*

**Jak odczytać zrobioną konfigurację w MT:**
*   `/interface vlan print` (Wyświetla przypięcia VLAN do interfejsu)
*   `/ip address print` (Wyświetla adresy IP)
*   `/ip route print` (Weryfikacja routingu)

---

## 3. Skonfiguruj przełącznik
Zgodnie z wytyczną `B` (`README.md`), konfigurację oparto o urządzenia **MikroTik** działające np. z RouterOS na pokładzie (np seria CRS).

**Wymagania:**
*   IP: `10.10.1.123/24` przypisane do VLAN1. Brama: `10.10.1.1`.
*   VLAN 1: Port 1 nietagowany (dostęp), Port 4 tagowany (trunk)
*   VLAN 2: Port 2 nietagowany (dostęp), Port 4 tagowany (trunk)
*   VLAN 3: Port 3 nietagowany (dostęp), Port 4 tagowany (trunk)

**Rozwiązanie w MikroTik (WinBox / CLI Terminal):**
```routeros
# Utworzenie mostka i przypisanie portów 1-4 (tryb dostępowy PVID ułatwiający ruch):
/interface bridge add name=bridge1 vlan-filtering=no

/interface bridge port add bridge=bridge1 interface=ether1 pvid=1
/interface bridge port add bridge=bridge1 interface=ether2 pvid=2
/interface bridge port add bridge=bridge1 interface=ether3 pvid=3
/interface bridge port add bridge=bridge1 interface=ether4
# (Zostawienie domyślnego dla ether4 lub ręczna kontrola trunka)

# Konfiguracja tabeli VLAN
/interface bridge vlan add bridge=bridge1 vlan-ids=1 tagged=ether4 untagged=ether1
/interface bridge vlan add bridge=bridge1 vlan-ids=2 tagged=ether4 untagged=ether2
/interface bridge vlan add bridge=bridge1 vlan-ids=3 tagged=ether4 untagged=ether3

# Skonfigurowanie interfejsu VLAN1 na bridge'u do przypisania mu adresu zarządzania
/interface vlan add interface=bridge1 vlan-id=1 name=vlan1_mgmt
/ip address add address=10.10.1.123/24 interface=vlan1_mgmt
/ip route add dst-address=0.0.0.0/0 gateway=10.10.1.1

# Włączenie filtrowania (odcinając niepasujące reguły)
/interface bridge set bridge1 vlan-filtering=yes
```

**Jak odczytać zrobioną konfigurację (weryfikacja):**
*   `/interface bridge vlan print` (Tabela z otagowaniem)
*   `/interface bridge port print` (PVID dostępowe)
*   `/ip address print` (Adres zarządzania)

---

## 4. Połączenie urządzeń

Na podstawie konfiguracji z punktów wyżej (zdefiniowanych interfejsach VLAN na routerze i tagowanych/nietagowanych portach Switcha) łączymy sprzęt w następujący sposób:
1. Port drugi Rutera (skonfigurowany wyżej na ether2) jest połączony jednym kablem z `portem 4` na Przełączniku (Tworzy to magistralę Trunk przesyłającą tagowane ramki VLAN).
2. Przewodowy interfejs LAN1 serwera jest podłączony do **portu 1** Przełącznika (Sieć VLAN 1 / 10.10.1.0/24).
3. Przewodowy interfejs LAN2 serwera jest podłączony do **portu 2** Przełącznika (Sieć VLAN 2 / 10.10.2.0/25).
4. Przewodowy interfejs Stacji roboczej (LAN3) jest podłączony do **portu 3** Przełącznika (Sieć VLAN 3 / 10.10.3.0/26).

---

## 5. Montaż zapasowego dysku twardego

**Zasady bezpieczeństwa (BHP) podczas montażu:**
1. Stacja robocza musi zostać fizycznie odłączona od zasilania sieciowego (wypiąć kabel 230V).
2. Rozładuj swój potencjał antystatyczny poprzez dotknięcie obudowy.
3. Zdejmij boczny panel obudowy komputera stacjonarnego.
4. Zamontuj udostępniony dysk (HDD/SSD) do zatoki 3.5" lub 2.5" w obudowie komputera, korzystając z zaczepów/sanek lub przykręcając go śrubkami z użyciem śrubokręta.
5. Poprowadź kabel zasilający złącza SATA z zasilacza komputera i wepnij go do dysku.
6. Poprowadź i podłącz kabel transmisyjny SATA (data) między odpowiednim portem na płycie głównej komputera a zainstalowanym dyskiem.
7. Zgłoś Przewodniczącemu podniesieniem ręki chęć do pokazania poprawności instalacji.
8. Po sprawdzeniu, zamknij obudowę i uruchom system operacyjny komputera.

---

## 6. Identyfikacja parametrów (Zapasowy dysk)

Instrukcja z zadania narzuca środowisko Windows oraz program `CrystalDiskInfo`. Poniżej opracowano również Wariant C2 narzucony w wytycznych dla systemów stacji roboczych z `README.md`.

### Wariant C1 - System Windows (Domyślny z arkusza)
1. Zlokalizuj folder z narzędziami podany w poleceniu (`DOKUMENTACJA/PROGRAMY\DIAGNOSTYKA`), zainstaluj lub jeśli to wersja przenośna (Portable) uruchom program `CrystalDiskInfo`.
2. Z głównego menu okna programu, w górnym pasku upewnij się, że masz wybrany drugi zainstalowany dysk (oznaczony ikoną drugiego napędu).
3. Odczytaj wymagane informacje:
   - **Numer seryjny** (Serial Number) wyświetla się jako ciąg znaków po prawej stronie.
   - **Tryb interfejsu** (Interface / Transfer Mode) podany jest po lewej stronie, np. SATA / Serial ATA, lub standard PCIe.
   - **Temperatura** widoczna jako duża ikona w głównym panelu dysku.
   - **Czas pracy dysku [h]** (Power On Hours) wyświetlony jest po prawej stronie.
4. Wykonaj zrzut całego okna CrystalDiskInfo (przez PrintScreen lub systemowe Wycinanie), zapisz go na wskazanym USB pod nazwą `paramerty_dysku.jpg` i wypełnij Tabelę 1 (znajduje się ona pod koniec tego dokumentu).

### Wariant C2 - System Linux Ubuntu
Gdyby stacja wyposażona była w środowisko Linux (zgodnie z instrukcją C/README), diagnostyki programem pokroju CrystalDiskInfo można dokonać narzędziami wbudowanymi `smartctl` lub w graficznym menedżerze Dysków GNOME.
1. Otwórz terminal i wykorzystaj system S.M.A.R.T: `sudo smartctl -a /dev/sdb` (lub odpowiedni napęd np. `/dev/nvme0n1`).
2. W otrzymanym potężnym wyniku znakowym odszukaj:
   - `Serial Number:` z sekcji Device Model.
   - Interfejs `SATA` z sekcji `Transport` (lub `SATA Version is...`).
   - Temperaturę dysku np. `Temperature_Celsius` w liście atrybutów SMART (wartość w kolumnie RAW_VALUE).
   - Ilość godzin dysku jako atrybut oznaczony `Power_On_Hours`.
3. Zrób zrzut terminala lub aplikacji dyskowej i skopiuj pod zadaną w punkcie wyżej nazwą pliku na pamięć masową przenośną.

---

## 7. Skonfiguruj serwer (Windows Server / Ubuntu Server)

Zadanie narzuca na serwer Microsoft Windows, na którego potrzeby opisano rozwiązanie D1. Ze względu na regułę D w README, zaprezentowano też metodę dla wariantu D2 (Linux Server).

### Wariant D1 - System Windows Server (Domyślny dla tego zadania)

1. **Konfiguracja sieci (LAN1, LAN2):**
   - Skrótem `Win+R` wywołaj `ncpa.cpl`. Zmień nazwy dwóch kart sieciowych według schematu w zadaniu odpowiednio na `LAN1` i `LAN2`.
   - Zmień właściwości IPv4 połączenia **LAN1**: zaznacz stały adres, wpisz: IP: `10.10.1.10`, Maska podsieci: `255.255.255.0` (dla /24), Brama domyślna: `10.10.1.1`. Serwer DNS zmień na Preferowany `8.8.8.8` oraz Alternatywny `1.1.1.1`.
   - Zmień właściwości IPv4 połączenia **LAN2**: zaznacz stały adres, wpisz: IP: `10.10.2.20`, Maska podsieci: `255.255.255.128` (dla /25 wpisz 128 na końcu zamiast 0). Brama domyślna: `10.10.2.1`. Serwer DNS tak jak na drugiej karcie `8.8.8.8` i `1.1.1.1`.

2. **Przygotowanie Plików i Katalogów:**
   - W Eksploratorze dysków stwórz nowy folder `C:\ftp`.
   - W środku tego folderu, utwórz nowy plik tekstowy `egzamin.txt` i wpisz do niego na pierwszej linijce żądany tekst: `EGZAMIN ZAWODOWY INF.02`. Zapisz.

3. **Utworzenie użytkownika lokalnego Egzamin:**
   - Wejdź w aplet `Zarządzanie komputerem` (`compmgmt.msc`).
   - Przejdź do zakładki *Użytkownicy i grupy lokalne* -> Użytkownicy. Kliknij prawym w puste okno i wybierz `Nowy użytkownik`.
   - Nazwa Użytkownika: `Egzamin`. Hasło: `Zdajacy!@#`.
   - Zaznacz wymagane obostrzenia: Oznacz ptaszkiem (Enable) -> "Użytkownik nie może zmienić hasła" oraz "Hasło nigdy nie wygasa".

4. **Instalacja Rol (IIS z usługą FTP):**
   - Otwórz Menedżer Serwera (Server Manager) i przejdź do "Dodaj role i funkcje".
   - Przeklikaj do wyboru ról serwera, tam zaznacz `Serwer sieci Web (IIS)`. Kliknij dalej do "Usługi ról" serwera Web, rozwiń drzewko Serwer FTP i dodaj ptaszka przy **Usługa FTP** (FTP Service) i Usługa rozszerzenia FTP. Zakończ Instalację.

5. **Konfiguracja witryny FTP:**
   - Otwórz `Menedżer internetowych usług informacyjnych` (`inetmgr.msc`). Rozwiń pozycję serwera.
   - Prawym przyciskiem myszy na węzeł "Witryny" i wybierz **Dodaj witrynę FTP...**.
   - Nazwa witryny FTP: `FTP_egzamin`. Ścieżka fizyczna: `C:\ftp`. Dalej.
   - Powiązanie i SSL: W adresie IP upewnij się by otworzyć listę i ustawić go statycznie na `10.10.2.20`. Zaznacz SSL na "Brak protokołu SSL". Dalej.
   - Uwierzytelnianie i autoryzacja: (Można to zdefiniować z poziomu tego kreatora lub z poziomu ikonek IIS). Wybierz Uwierzytelnianie Anonimowe oraz Podstawowe. Zezwól na dostęp: Wszyscy Użytkownicy (lub wpisz grupy ręcznie w module autoryzacji potem) - zakończ.
   - Będąc w utworzonej już witrynie, przejdź w panelu centralnym do **Reguły autoryzacji FTP**. Powinny istnieć dwa oddzielne wpisy (albo skasuj to co narobił kreator i ustaw jeszcze raz na czysto wg polecenia):
      * Kliknij *Dodaj regułę zezwalającą*: Zaznacz opcję *Określeni użytkownicy* -> Wpisz `Egzamin`. Poniżej w opcjach zaznacz ptaszkiem pola *Odczyt* i *Zapis*. Kliknij OK.
      * Kliknij drugi raz *Dodaj regułę zezwalającą*: Wybierz opcję *Użytkownicy anonimowi*. Poniżej w opcjach zaznacz tylko *Odczyt*. Kliknij OK.

### Wariant D2 - System Linux Ubuntu Server

Jeżeli z wylosowanych maszyn na egzaminie padnie na Ubuntu (Reguła D) instrukcja wygląda jak niżej:

1. **Konfiguracja sieci:**
   - Wyedytuj plik np: `sudo nano /etc/netplan/00-installer-config.yaml`.
   - Zmień konfigurację portów (na stacji roboczej np. fizyczne porty to `enp0s3` jako LAN1, `enp0s8` jako LAN2).
     ```yaml
     network:
       version: 2
       ethernets:
         enp0s3:
           addresses: [10.10.1.10/24]
           routes:
             - to: default
               via: 10.10.1.1
           nameservers:
             addresses: [8.8.8.8, 1.1.1.1]
         enp0s8:
           addresses: [10.10.2.20/25]
           routes:
             - to: default
               via: 10.10.2.1
           nameservers:
             addresses: [8.8.8.8, 1.1.1.1]
     ```
   - *Uwaga:* Posiadanie dwóch włączonych bram domyślnych w netplan może powodować problemy w linuksie jeśli nie włączysz policy routingu (Source Routing). Dla bezpieczeństwa warto zakomentować default gateway w `enp0s8` i użyć statycznej marszruty `routes: - to: 10.10.0.0/16 via: 10.10.2.1` albo nie wpisywać jej wcale jeśli wszystkie VLANy spina ruter. W każdym razie standardowa konfiguracja egzaminacyjna to dwa oddzielne default routes i metryka).
   - Wdróż profil komendą `sudo netplan apply`.

2. **Przygotowanie Plików i Konto Użytkownika:**
   - Stwórz środowisko poleceniem: `sudo mkdir /ftp` oraz wpisz tekst `sudo bash -c 'echo "EGZAMIN ZAWODOWY INF.02" > /ftp/egzamin.txt'`.
   - Użytkownika stworzysz poleceniem: `sudo useradd -d /ftp -s /bin/bash Egzamin`. I hasło: `echo "Egzamin:Zdajacy!@#" | sudo chpasswd`.
   - Obostrzenia konta: Linux nie posiada prostej koncepcji "użytkownik nie może zmienić hasła" jak Windows (ustawia się parametry życia hasła w plikach shadow):
     `sudo chage -m 99999 Egzamin` (zapobiega zmianie), oraz max czas hasła `sudo chage -M 99999 Egzamin` (hasło nigdy nie wygasa).

3. **Konfiguracja Serwera (np vsftpd):**
   - Wpisz `sudo apt install vsftpd`.
   - Wyedytuj plik demona `sudo nano /etc/vsftpd.conf`:
     ```text
     listen=YES
     listen_ipv6=NO
     listen_address=10.10.2.20
     listen_port=21
     anonymous_enable=YES
     anon_root=/ftp
     local_enable=YES
     write_enable=YES
     chroot_local_user=YES
     local_root=/ftp
     allow_writeable_chroot=YES
     ```
   - Skonfiguruj prawa: Aby vsftpd działało jako publiczny read/write i logowanie ze specyficznym kontem (z reguły user Egzamin), nadaj prawa do `/ftp`: `sudo chown Egzamin:Egzamin /ftp` oraz zrestartuj demona `sudo systemctl restart vsftpd`.

---

## 8. Skonfiguruj stację roboczą i wykonaj testy FTP

Zadanie narzuca konfigurację na systemie Windows, jednak z uwzględnieniem `README.md` rozpisano opcję na Wariant C2 (Ubuntu).

### Wariant C1 - System Windows Desktop (Domyślny dla tego zadania)
1. **Konfiguracja LAN3:**
   - Wywołaj ustawienia kart z paska skrótu `ncpa.cpl`. Zmień nazwę karty przewodowej na `LAN3`.
   - Zmień właściwości IPv4 połączenia na statyczne, wpisz IP: `10.10.3.44`, Maska podsieci: `255.255.255.192` (dla formatu /26 końcówka to 192), Brama domyślna: `10.10.3.1`. Serwer DNS preferowany `8.8.8.8` oraz `1.1.1.1`.
2. **Działania na FTP z poziomu stacji:**
   - Włącz wbudowany systemowy Menedżer (Eksplorator) plików. W pasku adresu u góry wklej ścieżkę do serwera i wciśnij Enter: `ftp://10.10.2.20`.
   - Poprawnie skonfigurowany serwer ukaże ci widok anonimowy katalogu `/ftp`. Powinieneś tam widzieć plik `egzamin.txt`.
   - Aby się poprawnie zalogować kliknij Prawy Przycisk Myszy (na wolne pole) i wybierz "Zaloguj jako...".
   - Jako użytkownik wklep `Egzamin`, hasło to `Zdajacy!@#`.
   - Jeżeli serwer skonfigurowano poprawnie z prawami Zapisuj dla usera (Write), możesz na oknie folderu kliknąć PPM i wybrać Nowy -> Folder i nadać mu nazwę z zadania czyli `INF.02`.

### Wariant C2 - System Linux Ubuntu Desktop
1. **Konfiguracja LAN3:**
   - Otwórz w GUI ustawienia Sieci (Wired Connection/Połączenie Przewodowe), edytuj je nazywając `LAN3` i na karcie IPv4 zmień na Ręczne (Manual). IP na `10.10.3.44`, maskę zostaw jako `/26` lub wpisz `255.255.255.192`. Bramę na `10.10.3.1`, a DNS wpisz w pola po przecinku: `8.8.8.8, 1.1.1.1`. Zapisz profil.
2. **Działania na FTP z poziomu stacji:**
   - Otwórz menedżer plików ("Pliki" GNOME Nautilus), wejdź w opcję "Inne miejsca" i u dołu strony włącz łączenie z serwerem i wpisz: `ftp://10.10.2.20`. Po wyświetleniu monitu logowania wpisz Egzamin i hasło lub zrób to w pasku: `ftp://Egzamin:Zdajacy!@#@10.10.2.20`.
   - Utwórz katalog pod prawym Przyciskiem Myszy w wyrenderowanym graficznie systemie plików FTP.

---

## 9. Wykonaj testy komunikacji stacji z Serwerem i Przełącznikiem

Otwórz wiersz polecenia `cmd` (Wariant C1 - Win) lub Terminal (Wariant C2 - Linux) by spingować wskazane cele.

1. **Test z adresem w LAN1 serwera:** `ping 10.10.1.10`
2. **Test z adresem w LAN2 serwera:** `ping 10.10.2.20`
3. **Test z adresem w Przełączniku:** `ping 10.10.1.123`

*Rozwiązywanie problemów (komunikacja obcięta/timeout):* W architekturze z zadania mamy stację zapiętą do VLAN3 pod Windows, wykonującą pingi do urządzeń w innej sieci tj. VLAN1/VLAN2 połączonymi za pomocą routingu na MikroTiku. Zapora z wariantu D1 domyślnie blokuje wejściowe połączenia pingu ze stref zewnętrznych. Należy na Serwerze odpalić aplikację Zapory Defendera (`wf.msc`), z reguł wejściowych włączyć "Udostępnianie plików i drukarek (żądanie echa - ruch przychodzący ICMPv4)".

---

## Tabele egzaminacyjne

W oparciu o wykonane badanie za pomocą środowiska `CrystalDiskInfo` (dla wariantu C1) lub S.M.A.R.T (dla C2), uzupełnij przygotowaną niżej tabelkę egzaminacyjną. Podane wartości w tabeli poniżej są domyślnie testowe w odniesieniu do odczytów ze wirtualnego/szkoleniowego dysku SATA.

### Tabela 1. Parametry dysku stacji roboczej

| Lp. | Parametry | Wartość |
|-----|-----------|---------|
| 1. | Numer seryjny | `WD-WCC6Y6XU02K` (Zależnie od podpiętego fizycznie dysku) |
| 2. | Tryb interfejsu | `SATA/600` (dla dysku wpiętego kablem SATA do płyty, lub NVM Express dla m.2) |
| 3. | Temperatura [ºC] | `31 °C` (Zależnie od warunków SMART) |
| 4. | Czas pracy dysku [h] | `5674 h` (Podać ilość w odniesieniu do informacji Power_On_Hours) |
