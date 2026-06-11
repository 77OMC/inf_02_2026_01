# Rozwiązanie egzaminu INF.02-01-26.03-SG

**Wersja arkusza:** SG
**Sesja:** Czerwiec 2025 (Zgodnie z poleceniem)

---

## 1. Wykonaj na stacji roboczej montaż zapasowej pamięci RAM

**Opis wykonania:**
Należy przestrzegać podstawowych zasad BHP przy pracy z podzespołami:
1. Odłącz komputer od zasilania sieciowego (wyjmij kabel).
2. Rozładuj ładunki elektrostatyczne ze swojego ciała (dotykając uziemionego metalowego elementu lub zakładając opaskę anty-ESD).
3. Zdejmij panel boczny obudowy stacji roboczej.
4. Odciągnij plastikowe zatrzaski na bocznych krawędziach gniazd pamięci DIMM, po czym wyciągnij zamontowaną tam kość pamięci RAM.
5. Zamontuj zapasową kość pamięci RAM uważając na położenie wcięcia w slocie. Po dociśnięciu zatrzaski zamkną się automatycznie.
6. Zgłoś gotowość Przewodniczącemu ZN przez podniesienie ręki do oceny montażu.
7. Po zgodzie, zamknij obudowę i uruchom stację roboczą (Windows).

---

## 2. Identyfikacja parametrów systemu i podzespołów komputera

Instrukcja wymaga odszukania następujących parametrów w systemie dla Tabeli 1: `Nazwa systemu`, `Oznaczenie wersji systemu`, `Rozmiar zainstalowanej RAM`, `Rozmiar dostępnej RAM`, `Rozmiar dostępnej pamięci wirtualnej`, `Obszar pliku stronicowania`. Zrzuty mają trafić do folderu `podzespoly` na wskazanym USB. Poniżej warianty C1 i C2.

### Wariant C1 - System Windows (Domyślny z arkusza)
W systemie Windows najwygodniej posłużyć się wbudowanym programem **Informacje o systemie**.
1. Naciśnij skrót klawiszowy `Win + R`, a następnie wpisz: `msinfo32` i naciśnij Enter.
2. W oknie w podsumowaniu systemu widoczne będą na pierwszych linijkach wszystkie wymagane parametry, m.in.:
   - **Nazwa systemu operacyjnego** (OS Name) - np. *Microsoft Windows 10 Pro*
   - **Wersja** (Version) - np. *10.0.19045 kompilacja 19045*
   - **Zainstalowana pamięć fizyczna (RAM)** (Installed Physical Memory) - np. *16,0 GB*
   - **Całkowita pamięć fizyczna** i **Dostępna pamięć fizyczna** (Available Physical Memory) - np. *7,53 GB*
   - **Dostępna pamięć wirtualna** (Available Virtual Memory) - np. *11,2 GB*
   - **Obszar pliku stronicowania** (Page File Space) - np. *2,38 GB*
3. Wykonaj zrzut ekranu okna msinfo32 (np. wciskając `PrtScn` i wklejając do Painta), nazwij go sensownie i zapisz w folderze `Egzamin-X\podzespoly`. Odczytane dane wpisz do arkusza do Tabeli 1 (wzór poniżej w dokumencie).

### Wariant C2 - System Linux Ubuntu
W środowisku Ubuntu polecenia terminala wskażą odpowiednie wartości.
1. Otwórz okno terminala i użyj następujących komend:
   - **Nazwa i Wersja systemu operacyjnego:** Wpisz `cat /etc/os-release` lub `lsb_release -a`. Pokaże to np. *Ubuntu 22.04 LTS*.
   - **Zainstalowana i Dostępna pamięć RAM:** Wpisz polecenie `free -m` lub `free -h`.
     - Kolumna `total` w wierszu `Mem` wskazuje *zainstalowaną pamięć*.
     - Kolumna `available` w wierszu `Mem` wskazuje *dostępną pamięć RAM*.
   - **Pamięć wirtualna / Obszar pliku stronicowania (SWAP):** Parametry w Linuxie określają to mianem SWAP.
     - Komenda `free -h` podaje również przestrzeń `Swap` (odpowiednik pliku stronicowania w Win), w tym `total` (obszar pliku stronicowania/partycji) i `free` (dostępna).
     - Dla detali można użyć `swapon -s` (wielkość i typ miejsca wymiany). W Linuksie pamięć wirtualna to z reguły logiczne połączenie RAM i Swap (choć formalnie podaje się wartości samego SWAP jako odpowiednik "pliku stronicowania").
2. Zrób zrzut ekranu widoku terminala lub odpowiednich okien z komend (np. wciskając PrtScn) i zachowaj w wymaganym na pendrive folderze.

---
## 3. Skonfiguruj ruter
Zgodnie z wytyczną `B` (`README.md`), konfigurację oparto o urządzenia **MikroTik**.

**Wymagania:**
*   Interfejs WAN IP: `212.10.10.20/26`
*   Brama domyślna WAN: `212.10.10.1`
*   Serwery DNS: `8.8.8.8` oraz `8.8.4.4`
*   Interfejs LAN IP: `192.168.0.1/24`
*   DHCP włączony z zakresem: `192.168.0.60 ÷ 192.168.0.80`

**Rozwiązanie w MikroTik (WinBox / CLI Terminal):**
```routeros
# Czyszczenie ustawień (opcjonalne, dla środowiska egzaminacyjnego zależnie od reguł):
/system reset-configuration no-defaults=yes skip-backup=yes

# Ustawienie adresacji dla WAN (przyjmując że WAN to ether1):
/ip address add address=212.10.10.20/26 interface=ether1

# Ustawienie bramy domyślnej rutera wychodzącej na świat:
/ip route add dst-address=0.0.0.0/0 gateway=212.10.10.1

# Ustawienie serwerów DNS na ruterze z opcją nasłuchiwania dla klientów LAN:
/ip dns set servers=8.8.8.8,8.8.4.4 allow-remote-requests=yes

# Ustawienie adresacji dla LAN (przyjmując np. ether2 podłączony do switcha):
/ip address add address=192.168.0.1/24 interface=ether2

# Ustawienie puli i uruchomienie serwera DHCP:
/ip pool add name=dhcp_pool1 ranges=192.168.0.60-192.168.0.80
/ip dhcp-server add name=dhcp1 interface=ether2 address-pool=dhcp_pool1 disabled=no
# Ustalenie domyślnej sieci dla klientów pobierających IP z DHCP
/ip dhcp-server network add address=192.168.0.0/24 dns-server=192.168.0.1 gateway=192.168.0.1
```

**Jak odczytać zrobioną konfigurację w MT:**
*   `/ip address print` (IP rutera)
*   `/ip route print` (Trasy routingu)
*   `/ip dns print` (Weryfikacja adresów DNS)
*   `/ip dhcp-server print` oraz `/ip pool print` (Weryfikacja usługi).

---

## 4. Skonfiguruj przełącznik
Zgodnie z wytyczną `B` (`README.md`), konfigurację oparto o urządzenia **MikroTik**. Poniżej zakładamy switch pracujący na bazie RouterOS.

**Wymagania:**
*   Adres IP: `192.168.0.2/24`
*   Brama: IP rutera w LAN (`192.168.0.1`)

**Rozwiązanie w MikroTik (WinBox / CLI Terminal):**
```routeros
# Jeżeli switch pracuje jako bridge w RouterOS (np porty dostępowe ether1-ether24 są spięte w bridge1):
/ip address add address=192.168.0.2/24 interface=bridge1

# Ustawienie bramy do zarzadzania
/ip route add dst-address=0.0.0.0/0 gateway=192.168.0.1
```

**Jak odczytać zrobioną konfigurację w MT:**
*   `/ip address print`
*   `/ip route print`

---

## 5. Połączenie urządzeń

Zgodnie ze schematem oraz powyższą adresacją:
1. Skrętka UTP wpinana w gniazdo E-X oraz do portu zewnętrznego na ruterze (WAN, np. ether1).
2. Kabel do punktu LAN1 (z interfejsu lokalnego rutera np. ether2) idzie do pierwszego portu przełącznika.
3. Karta sieciowa (interfejs pierwszy) stacji roboczej połączona z przełącznikiem (np. do 2 portu).
4. Przełącznik połączony kablem z drukarką sieciową na stanowisku.
5. Pierwszy przewodowy interfejs serwera podłączony kablem do przełącznika (np. 3 portu).

---

## 6. Na stacji roboczej skonfiguruj system Windows (Interfejsy, użytkownicy, drukarka)

Zadanie narzuca z góry system Windows na stacji, niemniej na potrzeby dokumentacji dla wytycznych C z `README.md`, zostają rozpisane oba warianty na równi.

### Wariant C1 - System Windows (Domyślny dla tego zadania)

1. **Konfiguracja adresacji (ncpa.cpl):**
   - Wejdź we Właściwości karty sieciowej podłączonej do przełącznika -> IPv4.
   - Wybierz *Użyj następującego adresu IP*.
   - Wpisz Adres IP: `192.168.0.10+X` (gdzie X to nr. stanowiska, np. dla st nr 1 to `192.168.0.11`).
   - Maska: `255.255.255.0`
   - Brama domyślna: `192.168.0.1` (IP LAN rutera)
   - Preferowany serwer DNS: `192.168.0.1`

2. **Drukarka sieciowa (RAW, 192.168.0.200):**
   - Otwórz Ustawienia -> Drukarki i Skanery. (Albo ze starego apletu Panel Sterowania -> Urządzenia i Drukarki `control printers`).
   - Dodaj drukarkę. Użyj "Drukarki, której szukam nie ma na liście".
   - Wybierz "Dodaj drukarkę używając adresu TCP/IP lub nazwy hosta".
   - Typ urządzenia to `Urządzenie TCP/IP`, podaj Hostname/IP address: `192.168.0.200`. Odznacz ewentualne zapytania po auto.
   - Jeżeli wykryje kartę sieciową i poprosi o typ, wybierz Custom/Niestandardowy, kliknij Ustawienia i upewnij się, że Protokół to **RAW** z domyślnym portem 9100. Załaduj wytypowane sterowniki i po instalacji wybierz opcję drukuj stronę testową.

3. **Ustawienia złożoności haseł:**
   - Uruchom Zasady Zabezpieczeń Lokalnych wciskając `Win+R` i wpisując `secpol.msc`.
   - Przejdź pod gałąź *Zasady konta* -> *Zasady haseł*.
   - Zmień parametr "Hasło musi spełniać wymagania co do złożoności" na **Włączony** (Enabled).

4. **Konto użytkownika EGZAMIN:**
   - Otwórz wiersz poleceń `cmd` z uprawnieniami administratora lub aplet Zarządzania Komputerem `compmgmt.msc`.
   - Poprzez CMD wpisz komendę dodającą użykownika:
     `net user EGZAMIN INF@02egz# /add /fullname:"EGZAMIN" /passwordchg:no`
   - Następnie dodaj tego użytkownika do określonej lokalnej grupy bezpieczeństwa wpisując komendę w CMD:
     `net localgroup "Czytelnicy dzienników zdarzeń" EGZAMIN /add` (Event Log Readers).

### Wariant C2 - System Linux Ubuntu Desktop
Oto te same operacje co punkt wyżej gdyby stacja zadana przez organizatora posiadała środowisko Linuksowe.

1. **Konfiguracja IP:**
   - W Ustawieniach graficznych Sieci (GNOME GUI), wejdź we właściwości przewodowe i w zakładce IPv4 zamiast metody DHCP wybierz **Ręczne (Manual)**.
   - Wpisz IP: `192.168.0.10+X`, Maska z rozwijanego lub wpisz `255.255.255.0`, Brama: `192.168.0.1`.
   - Pole DNS: `192.168.0.1`.
   - Wyłącz i włącz suwak od danej karty. Odczyt komendą: `ip a`.

2. **Instalacja Drukarki (RAW / TCP IP):**
   - Otwórz interfejs Ustawienia -> Drukarki -> Dodaj drukarkę.
   - W polu wyszukiwania lub adresowym na liście po rozwinięciu wpisz adres bezpośredni ipp/raw np: `socket://192.168.0.200:9100` lub `ipp://192.168.0.200`. Zatwierdź i wejdź do opcji by wybrać wydruk testowy.

3. **Polityka złożoności i założenie użytkownika EGZAMIN:**
   - Hasło musi być złożone wg polecenia, w Linuksie do regulacji złożoności zazwyczaj używa się pakietu `libpam-pwquality`, ale z uwagi na brak internetu (lub dla jednego użytkownika), komendy tworzące wystarczą.
   - Wpisz do terminala: `sudo useradd -m -c "EGZAMIN" EGZAMIN` (konto)
   - Nadaj/ustaw hasło: `echo "EGZAMIN:INF@02egz#" | sudo chpasswd`
   - Odbierz prawo zmiany: Aby zakazać używania polecenia passwd (lub by je ograniczyć na czas minimalny), w poleceniu np. `sudo chage -m 9999 EGZAMIN` ustaw minimalny wiek hasła na kilkanaście lat w przód co zablokuje zmiany.
   - "Czytelnicy dzienników zdarzeń": Odpowiednikiem grupy logów np. dzienników journalctl w Linuxie jest dodanie użytkownika do grupy `adm` lub `systemd-journal`: `sudo usermod -aG systemd-journal EGZAMIN`.

---
## 7. Utwórz na stacji roboczej pliki wsadowe

Zadanie na tworzenie plików "wsadowych" `.bat` z arkusza wymusza przygotowanie ich naturalnie pod konsole `cmd.exe` systemu Windows, jednakowo wariantowo zostaną omówione skrypty Bash na wypadek implementacji na architekturze Linuxa.

### Wariant C1 - Pliki wsadowe Windows (Domyślny dla tego zadania)

Najpierw otwórz Wiersz polecenia i uruchom polecenia z zadania przygotowujące strukturę:
`mkdir C:\Wsadowe` i przekopiowanie trzech określonych tam folderów (`archiwum1`, `archiwum2`, `dane`) poleceniem kopiowania. (np: `xcopy /E /I D:\PLIKI\archiwum1 C:\Wsadowe\archiwum1\`).

**Plik 1: `C:\Wsadowe\plik1.bat`**
Skrypt wykonuje kopiowanie wszystkich plików z folderu dane do archiwum1. W trybie notatnika wpisz i zapisz kod:
```cmd
@echo off
copy "C:\Wsadowe\dane\*.*" "C:\Wsadowe\archiwum1\"
```

**Plik 2: `C:\Wsadowe\plik2.bat`**
Skrypt wykonuje kopiowanie danych plików po rozszerzeniu, które przychodzi jako argument `%1`.
```cmd
@echo off
if "%~1"=="" (
    echo "Nie podano rozszerzenia w argumencie!"
    goto :EOF
)
copy "C:\Wsadowe\dane\*.%~1" "C:\Wsadowe\archiwum2\"
```

**Plik 3: `C:\Wsadowe\plik3.bat`**
Ten skrypt dopisuje wynik na końcu istniejącego już pliku tekstowego `dane\informacje.txt`. Będzie wymagał systemowego wygenerowania daty i listy samego root (`C:\`).
```cmd
@echo off
echo Nowy wpis w dniu: >> "C:\Wsadowe\dane\informacje.txt"
echo %date% >> "C:\Wsadowe\dane\informacje.txt"
dir C:\ /b >> "C:\Wsadowe\dane\informacje.txt"
```

Zgodnie z poleceniem należy następnie uruchomić w terminalu na próbę te pliki, pamiętając by dla pliku 2 dopisać argument np. `plik2.bat txt`.

### Wariant C2 - Skrypty Bash dla Ubuntu (Implementacja zastępcza)

Zamiast .bat, pliki shellowe `.sh`. Przyjęty bazowy folder np `/Wsadowe` (z poziomu admina, lub `~/Wsadowe` dla usera).

**Plik 1: `plik1.sh`**
```bash
#!/bin/bash
cp -r /Wsadowe/dane/* /Wsadowe/archiwum1/
```

**Plik 2: `plik2.sh`**
```bash
#!/bin/bash
if [ -z "$1" ]; then
    echo "Brak parametru z rozszerzeniem!"
    return 1
fi
cp /Wsadowe/dane/*."$1" /Wsadowe/archiwum2/
```

**Plik 3: `plik3.sh`**
```bash
#!/bin/bash
echo "Nowy wpis w dniu:" >> /Wsadowe/dane/informacje.txt
date '+%Y-%m-%d' >> /Wsadowe/dane/informacje.txt
ls -1 / >> /Wsadowe/dane/informacje.txt
```

Przed ich uruchomieniem należy oczywiście najpierw nadać prawo do wykonywania wpisując `chmod +x plik1.sh plik2.sh plik3.sh`.

---
## 8. Skonfiguruj serwer z systemem operacyjnym Linux (wariant D2/D1)

Zadanie narzuca z góry system Linux na serwer. Niemniej zachowując instrukcję C/D z README, zostają przedstawione dwa środowiska z zastrzeżeniem, że D2 jest tutaj natywnym wymogiem arkusza.

### Wariant D2 - System Linux Ubuntu Server (Domyślny dla tego zadania)

1. **Konfiguracja sieci:**
   - Otwórz plik: `sudo nano /etc/netplan/00-installer-config.yaml`
   - Odszukaj interfejs przełącznika (np. `enp0s3`), oraz drugi wyłączony (np. `enp0s8`). Zapisz:
     ```yaml
     network:
       version: 2
       ethernets:
         enp0s3:
           addresses:
             - 192.168.0.30+X/24   # W miejscu X podstawiamy nr stanowiska np. 192.168.0.31
           routes:
             - to: default
               via: 192.168.0.1
           nameservers:
             addresses: [127.0.0.1]
         enp0s8:
           dhcp4: no
           link-local: []
     ```
   - Skonfiguruj wyłączenie drugiej karty (alternatywnie zrzucenie `ip link set enp0s8 down`). Następnie wprowadź profil komendą `sudo netplan apply`.

2. **Domyślna lokalizacja Serwera HTTP (Zapis dla Tabeli 2):**
   - Po instalacji serwera (`sudo apt install apache2` lub nginx) domyślnym katalogiem hostującym witryny HTML w Ubuntu i Debianach jest **`/var/www/html`**. Należy wpisać tę wartość do wiersza Tabeli 2.

3. **Główna Witryna i uprawnienia:**
   - Skopiuj plik z pendrive do głównej witryny nadpisując defaultowe ułożenie webu: `sudo cp /media/usb/PLIKI/index.html /var/www/html/index.html`.
   - Zrestartuj demona witryny HTTP `sudo systemctl restart apache2`.
   - Wyświetlenie w GUI (jeżeli serwer je posiada) lub w klienckiej maszynie C1/C2 (jeśli nie): otwórz przeglądarkę i wpisz `http://localhost/` lub adres IP z punktu wyżej.
   - Odczytaj grupę HTTP: Sprawdź z jakim userem włączony jest proces wpisując polecenie terminalowe `ps aux | grep apache2`. Wyszczególnionym w domyślnej konfiguracji użytkownikiem i grupą serwera na dystrybucjach typu Ubuntu jest zazwyczaj użytkownik `www-data` należący do grupy `www-data`.

4. **Katalog kopii i zmiana uprawnień (Chown, Chmod):**
   - Utwórz żądany katalog w głównym systemie plików: `sudo mkdir /kopie_strony`.
   - Ustaw prawa na 750 (zgodnie z: Właściciel: rwx [7], Grupa: r-x [5], Inni: --- [0]):
     `sudo chmod 750 /kopie_strony`
   - Zmień własność z root na odczytany proces serwera:
     `sudo chown www-data:www-data /kopie_strony`


### Wariant D1 - System Windows Server

Jeżeli z wylosowanych maszyn na egzaminie padnie na Windows Server (Wbrew logice z arkusza 03) instrukcja wygląda jak niżej:

1. **Konfiguracja sieci:**
   - Wykonaj polecenie wywołania ustawień: `ncpa.cpl`.
   - Na pierwszym interfejsie ustaw stały IP na `192.168.0.30+X` z maską `255.255.255.0`, bramą `192.168.0.1` oraz DNS `127.0.0.1`.
   - Drugi interfejs po prostu kliknij prawym i daj **Wyłącz** (Disable).

2. **Domyślna lokalizacja (Tabela 2):**
   - Po dodaniu roli `Server IIS` w Menedżerze Serwera, Windows operuje domyślną witryną WWW na katalogu: **`C:\inetpub\wwwroot`**.

3. **Główna witryna WWW, katalog i uprawnienia NTFS:**
   - Skopiuj plik HTML do `C:\inetpub\wwwroot\index.html`.
   - W usługach serwera WWW i w ogóle w Windows, odpowiednikiem procesu apache w Menedżerze zadań (`w3wp.exe`) są usługi `IIS_IUSRS` oraz konto procesowe aplikacji z DefaultAppPool ewentualnie konto `IUSR`.
   - Utwórz katalog dyskowy `C:\kopie_strony`.
   - Kliknij prawym Właściwości -> Zabezpieczenia -> Zaawansowane. Odznacz dziedziczenie usuwając starych użytkowników.
   - Wprowadź zasady jak wyżej: Dodaj uprawnienia "Właściciela - pełne": ustaw wybranego Właściciela folderu na koncie IIS_IUSRS (lub Właściciela konta administrator na nim z Pełną Kontrolą). Z kolei "Grupie": dodaj do ACL np grupę autoryzowanego serwera nadając tylko ptaszek przy `Odczyt i wykonanie`.

---
## 9. Wykonaj testy komunikacji serwera

W systemie serwera otwórz powłokę np terminal Bash lub cmd w Windows i wprowadź seryjnie polecenia sprawdzające odpowiedź ECHA od poszczególnych elementów topologii.
1. `ping 192.168.0.10+X` (Stacja robocza)
2. `ping 192.168.0.2` (Przełącznik)
3. `ping 192.168.0.200` (Drukarka IP)
4. `ping 192.168.0.1` (Interfejs wewnętrzny Rutera, chociaż zadanie wspomina LAN rutera to jako bramy do zpingowania)

**Korekta zapory sieciowej na Stacji (Zablokowany ping):**
*   Jeżeli jest to wariant Windows (Stacja C1): Należy wejść w Zaporę (Firewall with Advanced Security - `wf.msc`), w liście `Reguły Przychodzące` znajdź i włącz (Zezwól / Enable) regułę *Udostępnianie plików i drukarek (żądanie echa - ruch przychodzący ICMPv4)* aby host Windows nie zrzucał pakietów Ping.
*   Jeżeli jest to wariant Linux Ubuntu (Stacja C2): Host linux zazwyczaj bez problemu domyślnie odpowiada na ICMP (Ping). Jeśli mimo to blokuje go usługa Uncomplicated Firewall, to wpisz komendę z sudo: `sudo ufw allow icmp`.


---

## Tabele egzaminacyjne z poleceń

Zgodnie z wymaganiami punktu 2 (diagnoza msinfo32/lscpu) i 8 (serwer Apache/IIS), ujęto parametry w formie tekstowej tak, by przygotować zdającego na treść arkusza. (Podane wartości liczbowe dla przykładu domyślnie 16GB z Windows, resztę należy zweryfikować fizycznie odczytami)

### Tabela 1. Parametry systemu i podzespołów

| Parametr | Odczytana Wartość |
|----------|-------------------|
| Nazwa systemu operacyjnego | *Microsoft Windows 10 Pro / Ubuntu 22.04 LTS* |
| Oznaczenie wersji systemu operacyjnego | *10.0.19045 kompilacja 19045* |
| Rozmiar zainstalowanej pamięci RAM | *16,0 GB* |
| Rozmiar dostępnej pamięci RAM | *7,53 GB* |
| Rozmiar dostępnej pamięci wirtualnej | *11,2 GB (zależnie od wariantu C1/C2)* |
| Obszar pliku stronicowania | *2,38 GB (lub SWAP w Linuksie np 2 GB)* |

---

### Tabela 2. Lokalizacja głównej witryny serwera HTTP

Dla odpowiednio przypisanej architektury wpis należy skorelować z systemem, co daje następujące wpisy w rzędzie tabelki.

| Lokalizacja (dla Linuksa np. Apache): | `/var/www/html` |
|---------------------------------------|----------------|
| **Lokalizacja (dla Windows np. IIS):** | `C:\inetpub\wwwroot` |
