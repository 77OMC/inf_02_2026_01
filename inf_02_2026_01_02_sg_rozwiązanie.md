# Rozwiązanie egzaminu INF.02-02-26.01-SG

**Wersja arkusza:** SG
**Sesja:** Czerwiec 2025 (Zgodnie z poleceniem)

---

## 1. Wykonaj montaż okablowania sieciowego

**Zgodnie z wytycznymi w `README.md` (Punkt A):** Pominąłem zadanie dotyczące montażu okablowania sieciowego.
*(Zadanie polegało na zarobieniu kabla krosowego prostego w sekwencji T568B).*

---
## 2. Skonfiguruj ruter
Zgodnie z wytyczną `B` (`README.md`), konfigurację oparto o urządzenia **MikroTik**.

**Wymagania:**
*   Adres LAN IP: `192.168.1.1/24`
*   DHCP Włączony: zakres `192.168.1.101 ÷ 192.168.1.150`
*   Rezerwacja DHCP 1: `192.168.1.101` dla serwera.
*   Rezerwacja DHCP 2: `192.168.1.102` dla stacji roboczej.
*   Czas dzierżawy: `12 godzin`
*   Serwer DNS rozsyłany z DHCP: `192.168.1.101`

**Rozwiązanie w MikroTik (WinBox / CLI):**

1. Ustawienie IP dla portu (np. `ether2` połączonego do przełącznika):
   ```routeros
   /ip address add address=192.168.1.1/24 interface=ether2
   ```

2. Konfiguracja puli i serwera DHCP:
   ```routeros
   /ip pool add name=dhcp_pool1 ranges=192.168.1.101-192.168.1.150
   /ip dhcp-server add name=dhcp1 interface=ether2 lease-time=12h address-pool=dhcp_pool1 disabled=no
   /ip dhcp-server network add address=192.168.1.0/24 dns-server=192.168.1.101 gateway=192.168.1.1
   ```

3. Statyczne rezerwacje adresów IP z DHCP (należy najpierw odczytać MAC z urządzeń serwera i stacji roboczej):
   ```routeros
   /ip dhcp-server lease add address=192.168.1.101 mac-address=AA:BB:CC:DD:EE:FF server=dhcp1
   /ip dhcp-server lease add address=192.168.1.102 mac-address=11:22:33:44:55:66 server=dhcp1
   ```
   *(Zastępując AA:BB.. faktycznymi wartościami odczytanymi z systemu `ip a` lub `ipconfig /all`)*

**Weryfikacja / odczyt danych na urządzeniu:**
*   Sprawdzenie interfejsu LAN: `/ip address print`
*   Sprawdzenie sieci i opcji DHCP: `/ip dhcp-server network print`
*   Sprawdzenie statycznych rezerwacji (Dzierżawy): `/ip dhcp-server lease print`

---

## 3. Skonfiguruj przełącznik
Zgodnie z wytyczną `B` (`README.md`), konfigurację oparto o urządzenia **MikroTik**. Zakładamy zarządzalny przełącznik (np. z linii CRS lub CSS) działający pod kontrolą RouterOS.

**Wymagania:**
*   IP urządzenia: `192.168.1.2/24` (na VLAN o ID 2)
*   Brama: IP rutera (`192.168.1.1`)
*   VLAN 802.1Q włączony, ID=2.
*   Porty 1, 2, 3 w VLAN 2 nietagowane (dostępowe/access).

**Rozwiązanie w MikroTik (RouterOS CLI / WinBox):**

1. Utworzenie interfejsu `bridge` oraz obsługa VLAN:
   ```routeros
   # Tworzymy bridge bez domyślnie włączonego filtrowania VLAN aby nie odciąć się od zarządzania w złym momencie
   /interface bridge add name=bridge1 vlan-filtering=no

   # Dodajemy porty ether1, ether2 i ether3 do bridge'a jako porty dostępowe dla PVID 2
   /interface bridge port add bridge=bridge1 interface=ether1 pvid=2
   /interface bridge port add bridge=bridge1 interface=ether2 pvid=2
   /interface bridge port add bridge=bridge1 interface=ether3 pvid=2

   # Tworzymy logikę w tablicy VLAN, zdefiniowanie nietagowanych portów
   /interface bridge vlan add bridge=bridge1 vlan-ids=2 untagged=ether1,ether2,ether3

   # Dodanie wirtualnego interfejsu VLAN do zarządzania z routera dla switcha
   /interface vlan add interface=bridge1 vlan-id=2 name=vlan2

   # Włączamy filtrowanie po upewnieniu się, że konfig jest poprawny
   /interface bridge set bridge1 vlan-filtering=yes
   ```

2. Przypisanie adresu IP do interfejsu zarządzającego `vlan2`:
   ```routeros
   /ip address add address=192.168.1.2/24 interface=vlan2
   ```

3. Ustawienie bramy domyślnej:
   ```routeros
   /ip route add dst-address=0.0.0.0/0 gateway=192.168.1.1
   ```

**Weryfikacja / odczyt danych na urządzeniu:**
*   Sprawdzenie adresu IP: `/ip address print`
*   Sprawdzenie tablicy vlan-ów: `/interface bridge vlan print`
*   Sprawdzenie przydziału PVID na portach: `/interface bridge port print`

---

## 4. Połączenie urządzeń

Zgodnie z poleceniem adresami i fizycznymi opisami, podłączamy sprzęt przygotowanymi kablami:
1. Port LAN Rutera jest fizycznie połączony kablem UTP ze wskazanym przez schemat portem przełącznika (musi to być jeden z portów w tym samym VLANie 2 lub odpowiedni port trunk, tutaj domyślnie jedno z gniazd VLANu).
2. Druga Karta Sieciowa Serwera (LAN2) jest podłączona do **portu 1** Przełącznika.
3. Karta Sieciowa Stacji Roboczej (Linux LAN2) jest podłączona do jednego z portów wskazanych na schemacie (np. port 2 przełącznika).
4. Drukarka 192.168.0.200 musi być dostępna przez routing, więc jest wpięta w infrastrukturę dostępową w sieci wyższej/zewnętrznej (np. gniazdo E-X).

---

## 5. Skonfiguruj interfejsy sieciowe na serwerze

Zadanie narzuca z góry system Windows dla serwera, niemniej na potrzeby dokumentacji dla wytycznych README.md z podziałem C/D zostają stworzone opcje (D1 - Windows, D2 - Ubuntu Server).

### Wariant D1 - System Windows Server (Domyślny dla tego zadania)

1. Otwórz menedżer połączeń sieciowych używając klawiszy `Win+R` -> `ncpa.cpl`.
2. Zmień nazwy dwóch kart sieciowych według schematu odpowiednio na **LAN1** i **LAN2**.
3. **Konfiguracja LAN1:**
   - Kliknij Prawym Przyciskiem Myszy -> Właściwości -> Protokół IPv4.
   - Ustaw stały adres IP: `192.168.0.X` (gdzie X to podany numer stanowiska).
   - Ustaw maskę sieci na `255.255.255.0` (ponieważ `/24`).
   - Pole Bramy Domyślnej pozostaw **puste**.
   - Preferowany serwer DNS: `127.0.0.1` (localhost).
4. **Konfiguracja LAN2:**
   - Upewnij się, że we właściwościach protokołu IPv4 zaznaczone są opcje:
     - `Uzyskaj adres IP automatycznie`
     - `Uzyskaj adres serwera DNS automatycznie`.
5. **Odświeżenie dzierżawy DHCP:**
   - Otwórz wiersz polecenia (`cmd`) z uprawnieniami administratora i wpisz:
     ```cmd
     ipconfig /release
     ipconfig /renew
     ```
   - Otrzymasz adres `192.168.1.101` na interfejsie LAN2.

### Wariant D2 - System Linux Ubuntu Server

1. Otwórz plik z konfiguracją **Netplan**: `sudo nano /etc/netplan/00-installer-config.yaml`.
2. Wpisz następującą konfigurację modyfikując odpowiednio nazwy interfejsów sprzętowych (np. z `eth0`, `eth1` na `enp0s3`, `enp0s8`):
   ```yaml
   network:
     version: 2
     ethernets:
       LAN1:
         match:
           macaddress: "XX:XX:XX:XX:XX:XX"
         set-name: LAN1
         addresses: [192.168.0.X/24]
         nameservers:
           addresses: [127.0.0.1]
       LAN2:
         match:
           macaddress: "YY:YY:YY:YY:YY:YY"
         set-name: LAN2
         dhcp4: true
   ```
3. Zapisz plik i wprowadź w życie komendą `sudo netplan apply`.
4. Odświeżenie dzierżawy DHCP zrealizuj używając klienta np. `sudo dhclient -r LAN2` a następnie `sudo dhclient LAN2`.

**Odczyt parametrów / Weryfikacja dla systemu D1 / D2:**
*   D1: Komenda `ipconfig /all` do odczytu przypisanego adresu.
*   D2: Komenda `ip a show` do odczytu adresu oraz `cat /etc/resolv.conf` dla sprawdzenia DNS.

---

## 6. Skonfiguruj przewodowy interfejs sieciowy na stacji roboczej

Zadanie narzuca z góry system Linux, niemniej uwzględniając wymogi wytycznej **C** przygotowano warianty.

### Wariant C2 - System Linux Ubuntu Desktop (Domyślny)

1. Otwórz menedżera sieci, np. Ustawienia -> Sieć (GNOME GUI).
2. Zmień nazwę istniejącego połączenia sieciowego na `LAN2`.
3. Przejdź do zakładki IPv4. Ustaw metodę na **Automatyczną (DHCP)** (zazwyczaj domyślnie wybrana). Zapisz zmiany.
4. Otwórz terminal i odśwież dzierżawę: `sudo dhclient -r LAN2` oraz `sudo dhclient LAN2`. Ewentualnie zrestartuj interfejs w menedżerze `nmcli connection down LAN2 && nmcli connection up LAN2`. Z uwagi, że DHCP rutera zarezerwowało adres dla tego urządzenia, powinno ono pozyskać `192.168.1.102`.

### Wariant C1 - System Windows

1. Użyj skrótu `Win+R` i wpisz `ncpa.cpl`. Zmień nazwę karty sieciowej na **LAN2**.
2. Otwórz właściwości adaptera LAN2 -> `Protokół IPv4` i upewnij się że widnieje **Uzyskaj adres IP automatycznie** oraz to samo dla DNS.
3. Odśwież dzierżawę z Wiersza poleceń uruchomionego jako administrator: `ipconfig /release` a następnie `ipconfig /renew`.

**Odczyt / Weryfikacja dla systemu C1 / C2:**
*   C1 (Windows): Komenda `ipconfig /all`.
*   C2 (Linux): Komenda `ip addr show`.

---

## 7. Testy Komunikacji i adresy

1. Na Serwerze należy wywołać wiersz poleceń i spingować wymienne środowiska.
   Wariant D1 / D2 (Polecenie ping działa na obydwu):
   *   Do stacji roboczej: `ping 192.168.1.102`
   *   Do drukarki (LAN1): `ping 192.168.0.200`

   *Warunkiem jest odblokowanie w zaporze sieciowej stacji roboczej protokołu ICMPv4/Ping (jeśli wariant Windows) lub dodanie `ufw allow icmp` w Linuxie, by odpowiedzi docierały prawidłowo.*

2. Odczyt adresów na stacji i serwerze dla Egzaminatora:
   *   Serwer D1 (Win): `ipconfig /all`  (Powinno wyświetlić LAN1: 192.168.0.X, LAN2: 192.168.1.101).
   *   Serwer D2 (Lin): `ip addr show` lub `ifconfig`.
   *   Stacja C2 (Lin): `ip a show` lub `ifconfig` (Powinno wyświetlić LAN2: 192.168.1.102).
   *   Stacja C1 (Win): `ipconfig`.

---

## 8. Skonfiguruj serwer (DNS i WWW)

Poniżej przygotowano odpowiednie rozwiązanie podzielone na wytyczne serwerowe.

### Wariant D1 - System Windows Server (Domyślny dla tego zadania)

1. **Dodanie Ról:**
   - Otwórz `Menedżer serwera` (Server Manager), kliknij "Dodaj role i funkcje" i przeklikaj kreator. Zaznacz do instalacji opcje: **Serwer DNS** (DNS Server) oraz **Serwer sieci Web (IIS)** (Web Server IIS). Zakończ kreator instalacją.

2. **Konfiguracja Strefy DNS:**
   - Otwórz narzędzie `Menedżer DNS` (dnsmgmt.msc).
   - Rozwiń drzewo serwera, na gałęzi **Strefy wyszukiwania do przodu** kliknij PPM -> Nowa Strefa. Zaznacz strefę podstawową i wpisz nazwę `egzamin.local`.
   - Na utworzonej strefie kliknij PPM -> Nowy host (A). Wpisz nazwę hosta: `app`, adres IP interfejsu LAN2: `192.168.1.101`. Odznacz PTR (zrobimy to w osobnym kroku zgodnie z poleceniem). Zapisz.
   - Na gałęzi **Strefy wyszukiwania wstecznego** kliknij PPM -> Nowa Strefa. Wybierz IPv4 i jako identyfikator wpisz sieć LAN2 czyli `192.168.1`.
   - Przejdź do tej strefy, kliknij PPM -> `Nowy wskaźnik (PTR)`. W polu Host IP podaj host `101`, a w polu docelowej nazwy wskaż adres `app.egzamin.local.`.

3. **Kopiowanie folderu i ustawianie uprawnień NTFS:**
   - Zlokalizuj narzędzie i utwórz folder z wiersza poleceń lub za pomocą eksploratora: dysk `C:\app`.
   - Skopiuj do `C:\app` plik `index.html` z nośnika (np. dysk D).
   - Kliknij PPM na folder `C:\app`, Właściwości -> Zabezpieczenia -> Zaawansowane -> Wyłącz dziedziczenie (Usuń wszystkich użytkowników). Następnie dodaj grupę/użytkownika wpisując `Administratorzy`, nadaj im "Pełna kontrola". Dodaj także `IUSR` (konto od wbudowanego użytkownika IIS) zaznaczając tylko uprawnienia do "Odczyt i wykonywanie" (Oraz odczyt atrybutów itp.). Zatwierdź.

4. **Serwer IIS i nowa witryna WWW:**
   - Otwórz `Menedżer internetowych usług informacyjnych` (inetmgr.exe).
   - Rozwiń drzewo serwera -> Witryny. (Default Web Site można zatrzymać).
   - Kliknij PPM na `Witryny` -> `Dodaj witrynę sieci Web`.
   - Nazwa witryny: dowolna np. Egzamin.
   - Ścieżka fizyczna do folderu WWW: Wskaż `C:\app`.
   - Powiązanie (Binding) - Adres IP: `Wszystkie nieprzypisane` lub wprost `192.168.1.101`. Nazwa hosta: `app.egzamin.local`. Zakończ klikając OK.

### Wariant D2 - System Linux Ubuntu Server

W środowisku Linux proces trzeba poprowadzić inaczej używając Bind9 i Apache2.

1. **Instalacja Rol (pakietów):**
   - Wpisz: `sudo apt update && sudo apt install bind9 bind9utils bind9-doc apache2 -y`

2. **Konfiguracja Strefy DNS (Bind9):**
   - Edytuj główny plik `sudo nano /etc/bind/named.conf.local` i dodaj strefy:
     ```text
     zone "egzamin.local" {
         type master;
         file "/etc/bind/db.egzamin.local";
     };
     zone "1.168.192.in-addr.arpa" {
         type master;
         file "/etc/bind/db.192";
     };
     ```
   - Skopiuj domyślną bazę aby utworzyć bazę w przód: `sudo cp /etc/bind/db.local /etc/bind/db.egzamin.local`. Edytuj plik i wpisz zawartość:
     ```text
     $TTL    604800
     @       IN      SOA     ns.egzamin.local. admin.egzamin.local. (
                               1         ; Serial
                          604800         ; Refresh
                           86400         ; Retry
                         2419200         ; Expire
                          604800 )       ; Negative Cache TTL
     ;
     @       IN      NS      ns.egzamin.local.
     ns      IN      A       192.168.1.101
     app     IN      A       192.168.1.101
     ```
   - Skopiuj domyślną bazę do pliku strefy wstecznej `sudo cp /etc/bind/db.127 /etc/bind/db.192`. Edytuj tak samo parametry nagłówków SOA a w rekordach wpisz:
     ```text
     101     IN      PTR     app.egzamin.local.
     ```
   - Zrestartuj usługę: `sudo systemctl restart bind9`. Sprawdź status lub logi `systemctl status bind9`.

3. **Pliki i Witryna w Apache2:**
   - Utwórz katalog: `sudo mkdir /app`. Skopiuj tam z pendrive (punkt montowania /media/...) plik `index.html`. `sudo cp /media/usb/PLIKI/index.html /app`.
   - Edytuj konfig wirtualnego hosta w Apache2: `sudo nano /etc/apache2/sites-available/app.conf`
     ```xml
     <VirtualHost *:80>
         ServerName app.egzamin.local
         DocumentRoot /app

         <Directory /app>
             Options Indexes FollowSymLinks
             AllowOverride None
             Require all granted
         </Directory>
     </VirtualHost>
     ```
   - Aktywuj witrynę wyłączając domyślną: `sudo a2dissite 000-default.conf` i `sudo a2ensite app.conf`. Zrestartuj `sudo systemctl reload apache2`.

4. **Uprawnienia na dysku:**
   - Ogranicz dostępy rootowi dla Apache i innej grupie np sudo: `sudo chown -R root:www-data /app` oraz `sudo chmod -R 750 /app`. W ten sposób właściciel (np. serwer root) ma pełną kontrolę, serwer apache (www-data) użyje r-x czyli Odczytu i Wykonania.

### Działania na stacji klienckiej

**Dla C2 (Ubuntu Desktop) / Domyślnie z arkusza:** Uruchom przeglądarkę internetową, np. Mozilla Firefox, wpisz adres IP `http://192.168.1.101`. Strona `index.html` powinna się wyrenderować. (Dodatkowo za zgodnością serwera DNS można wpisać samo `http://app.egzamin.local/` i stacja odpyta ruter który wskaże serwer na IP końcówkę .101. Wpisanie DNS rozwiązuje problem rozpoznania VirtualHosta po nagłówku HTTP ServerName przez oprogramowanie WWW).
**Dla C1 (Windows Client):** Otwórz Edge/Chrome/Firefox i przetestuj wyżej podany adres URL.

---

## 9. Zamontuj zapasowy dysk na stacji roboczej

**Zasady bezpieczeństwa (BHP):**
Odłącz stację roboczą od sieci zasilającej. Rozładuj ładunki elektrostatyczne ze swojego ciała przed przystąpieniem do procedury.
Zdejmij boczną obudowę jednostki centralnej. Dysk umieść w dedykowanej zatoce komputera (kieszeni 2.5" lub 3.5"). Przykręć nośnik używając otworów bocznych by zapewnić stabilność. Do nośnika podłącz interfejs w postaci kabla SATA od strony Płyty Głównej oraz zasilający kabel SATA bezpośrednio z zasilacza (PSU). Zamknij obudowę i odpal system. Zgłoś fakt przewodniczącemu do sprawdzenia.

---

## 10. Skonfiguruj system Linux na stacji roboczej

Instrukcje te domyślnie należą do **Linuksa Ubuntu** (Zadanie narzuca środowisko wg arkusza, a zatem dla wytycznej C z README jest to wariant **C2**). Należy wykonać partycjonowanie z określonym formacie ext4, mount na `/dane` a potem sprawdzić wydajność.

### Wariant C2 - System Linux Ubuntu (Domyślny dla tego zadania)

1. **Tworzenie partycji z ext4:**
   - W terminalu wpisz `lsblk` aby zidentyfikować jaki zasób sprzętowy otrzymał właśnie nowy dysk np. to z reguły będzie literka `sdb`.
   - Wpisz `sudo fdisk /dev/sdb`. Wciśnij `n` by dodać nową partycję (wielkość domyślna z całego dysku - potwierdzaj Enterem od First Sector do Last Sector), następnie naciśnij `w` aby zapisać i wyjść.
   - Sformatuj ją narzędziem: `sudo mkfs.ext4 /dev/sdb1`.

2. **Montowanie w /dane:**
   - Utwórz katalog: `sudo mkdir -p /dane`.
   - Podmontuj: `sudo mount /dev/sdb1 /dane`. Weryfikacja: wpisz `df -h`.
   - *Opcjonalnie (jeśli wymagane utrzymanie stanu po restarcie): zrób wpis do `/etc/fstab` (np. podając `/dev/sdb1 /dane ext4 defaults 0 2`).*

3. **Testowanie z narzędzia (Disks - GNOME):**
   - Otwórz wbudowane graficzne narzędzie do obsługi dysków (Narzędzie: **Dyski** / Gnome-disks).
   - Kliknij na *dysk systemowy (sda)*. Wybierz pole zębatki / opcji nad nim i użyj "*Rozpocznij test wydajności partycji*". Wciśnij Zmień u góry po prawej by wklepać konfigurację: Liczba próbek = 200, Rozmiar = 20 MiB i Czas = 1000. Daj Uruchom tylko odczyt. Zmierzona zostanie **Średnia prędkość odczytu** w **MB/s**. Zanotuj wyniki do Tabeli 1. I stwórz screenshot.
   - Powtórz ten krok, klikając po lewej stronie w interfejsie na zapasowy, drugi podłączony dysk (sdb).
   - Wypełnij odpowiednio ostatnie pole w Tabeli 1.

### Wariant C1 - System Windows

Dla spełnienia zasad z README (paralelne wykonywanie tych samych działań w Windows Desktop), procedura jest następująca:

1. Systemy Microsoft domyślnie wykorzystują standard NTFS, exFAT lub ReFS do działania. Formatowanie zapasowego dysku do **ext4** byłoby możliwe np. używając komend w środowisku WSL 2 (`wsl --mount <DiskPath> --bare` itp) chociaż nie jest to natywne środowisko pulpitu Windows dla standardowych wolumenów danych, ew. format ten można zrealizować poprzez zewnętrzny program (np. AOMEI Partition Assistant / MiniTool). Biorąc pod uwagę warunki na egzaminie z brakiem internetu, zakładamy wykorzystanie `Zarządzanie dyskami (diskmgmt.msc)`, zainicjowanie dysku jako GPT/MBR i stworzenie "Nowy Wolumin Prosty" z domyślnie sformatowanym partycjonowaniem NTFS na pełną pulę po przypisaniu do niego dysku np D:\ pod folder (zamiast litery opcja - zamontuj w poniższym pustym folderze NTFS `C:\dane`).

2. **Test Wydajności:** Użyj domyślnego wiersza poleceń poprzez WinSat, np.: `winsat disk -drive c` do sprawdzenia prędkości (C - systemowego a potem `winsat disk -drive d` zapasowego). Z wyników (Data Read) uzupełnij prędkość odczytu sekwencyjnego do Tabeli 1.

---

## Tabela 1. Wydajność dysku

Zgodnie z wykonanymi testami w narzędziu (Zadanie 10):

| Partycja systemowa | Partycja na dysku zapasowym |
|--------------------|----------------------------|
| *Wpisz tu liczbę np.* **535,0** | *Wpisz tu liczbę np.* **540,5** |
| *Jednostka:* **MB/s** | *Jednostka:* **MB/s** |
| **Nazwa szybszego dysku (partycji)** | **Zapasowy dysk sdb (partycja na dysku zapasowym)** | *(Wskazać stosownie do uzyskanych pomiarów)*
