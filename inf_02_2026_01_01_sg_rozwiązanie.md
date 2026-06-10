# Rozwiązanie egzaminu INF.02-01-26.01-SG

**Wersja arkusza:** SG
**Sesja:** Czerwiec 2025 (Zgodnie z poleceniem)

---

## 1. Wykonaj na stacji roboczej wymianę pamięci RAM

**Opis wykonania:**
Zgodnie z zasadami BHP i instrukcją do zadania:
1. Należy odłączyć komputer od zasilania i zdjąć obudowę.
2. Przed ingerencją należy rozładować potencjał elektrostatyczny (np. dotykając niepomalowanego metalowego elementu obudowy lub zakładając opaskę antystatyczną).
3. Delikatnie odciągnąć zatrzaski na slotach pamięci RAM na płycie głównej, aby uwolnić obecną pamięć.
4. Wyjąć starą pamięć.
5. Przygotować zapasową kość pamięci z wyposażenia stanowiska. Zwrócić uwagę na wcięcie w złączu krawędziowym i odpowiednio dopasować do gniazda na płycie głównej.
6. Nacisnąć równomiernie z obu stron kości RAM do momentu, aż zatrzaski zabezpieczające zamkną się ze słyszalnym "kliknięciem".
7. *Zgłosić gotowość Przewodniczącemu ZN przez podniesienie ręki.*
8. Po uzyskaniu zgody, uruchomić system operacyjny Windows.

---

## 2. Przeprowadź diagnostykę stacji roboczej

Identyfikacja parametrów z Tabeli 2 wymaga odczytania informacji z systemu, a następnie ich porównania z wytycznymi z Tabeli 1 (dla emulacji systemu Android: Windows 10+, Core i3/Ryzen 3+, Wirtualizacja włączona, 8 GB RAM, dysk SSD).

Poniżej procedury sprawdzenia zarówno dla wariantu C1 (Windows) jak i C2 (Linux Ubuntu):

### Wariant C1 - System Windows

**Jak odczytać ustawienia i sporządzić zrzuty ekranu:**

1. **System operacyjny, Procesor, Pamięć RAM:**
   - Wywołaj aplikację **Ustawienia -> System -> Informacje** (Settings -> System -> About). Widać tam Edycję Windowsa, rodzaj i taktowanie procesora oraz wielkość RAM.
   - Alternatywnie z linii komend lub PowerShell:
     - `systeminfo`
     - `msinfo32` (wywołuje Narzędzie Informacje o systemie)
   - Zrób zrzut ekranu okna (np. za pomocą narzędzia Wycinanie - `Snipping Tool`).

2. **Karta graficzna:**
   - Wywołaj aplikację `dxdiag` (Narzędzie diagnostyczne DirectX). W zakładce **Ekran** widać Producenta karty graficznej.
   - Zrób zrzut ekranu.

3. **Wirtualizacja:**
   - Otwórz **Menedżer Zadań** (Task Manager - `Ctrl+Shift+Esc`), przejdź do zakładki **Wydajność** -> **Procesor CPU**. W prawym dolnym rogu będzie informacja "Wirtualizacja: Włączona/Wyłączona" (Virtualization: Enabled/Disabled).
   - Zrób zrzut ekranu.

4. **Dysk twardy (Pojemność, Rodzaj - HDD/SSD):**
   - Otwórz **Menedżer Zadań**, zakładka **Wydajność** -> **Dysk**. Będzie tam informacja o pojemności oraz podany typ: SSD lub HDD.
   - Alternatywnie wciśnij klawisz Windows, wpisz `Zarządzanie dyskami` (Disk Management) w celu odczytu fizycznej pojemności.
   - Zrób zrzut ekranu.

Wszystkie zrzuty należy zapisać w odpowiednim folderze (`Wymagania` na nośniku USB `Egzamin-x`).

### Wariant C2 - System Linux Ubuntu
*(Realizacja równoległa)*

**Jak odczytać ustawienia i sporządzić zrzuty ekranu:**
W systemie Ubuntu najszybciej skorzystać z terminala:

1. **System operacyjny:**
   - `cat /etc/os-release` lub `lsb_release -a` - pokaże nazwę systemu i wersję.
2. **Procesor i Wirtualizacja:**
   - `lscpu` - Wypisze model procesora, jego taktowanie oraz status wirtualizacji (sekcja `Virtualization features` - np. VT-x lub AMD-V). Włączenie wirtualizacji sprzętowej to flaga widoczna w procesorze (względem BIOS).
3. **Pamięć RAM:**
   - `free -h` - Wyświetli całkowitą ilość pamięci RAM w czytelnym formacie.
4. **Karta graficzna:**
   - `lspci | grep -i vga` lub `lshw -C display` - Zwróci informacje o producencie układu graficznego.
5. **Dysk twardy:**
   - `lsblk -o NAME,SIZE,TYPE,ROTA` - Komenda wyświetli wielkość i rodzaj bloku. Parametr `ROTA` oznacza obracające się dyski. `ROTA=0` to SSD (brak elementów mechanicznych), `ROTA=1` to HDD. Zwróci też pojemność.

Z wykonanych komend na terminalu należy wykonać zrzuty ekranu używając np. narzędzia `Przycisk PrintScreen` (Zrzut ekranu) i skopiować pliki do folderu `Wymagania` na USB.

**Podsumowanie oceny (dla obu wariantów):**
W wierszu `Ocena` w Tabeli 2 należy zapisać wniosek: "TAK, możliwe jest korzystanie z programu" LUB "NIE, nie jest możliwe". Trzeba zweryfikować czy CPU spełnia standard (minimum i3 lub Ryzen 3), RAM (minimum 8GB) oraz czy dysk to `SSD` i Wirtualizacja = `Włączona`.

---

## 3. Wykonaj montaż okablowania sieciowego

**Zgodnie z wytycznymi w `README.md` (Punkt A):** Pominąłem zadanie dotyczące montażu okablowania sieciowego.
*(Zadanie polegało na zarobieniu kabla U/UTP z obu stron wtykiem 8P8C według standardu T568B).*

---

## 4. Skonfiguruj ruter
Zgodnie z wytyczną `B` (`README.md`), poniżej znajduje się konfiguracja oparta o urządzenia **MikroTik**.

**Wymagania:**
*   WAN IP: `90.90.0.1/29`, brama: `90.90.0.2`
*   DNS na interfejsie WAN: `208.67.222.222`, `208.67.220.220`
*   LAN IP: `10.90.0.1/24`
*   Serwer DHCP: wyłączony

**Rozwiązanie w MikroTik (WinBox / CLI):**

*Aby ustawić te parametry w terminalu (New Terminal) MikroTika, wpisz:*
```routeros
# Reset ustawień fabrycznych (jeśli wymagany na początku pracy):
/system reset-configuration no-defaults=yes skip-backup=yes

# Ustawienie adresacji dla WAN (np. ether1):
/ip address add address=90.90.0.1/29 interface=ether1

# Ustawienie bramy domyślnej dla WAN:
/ip route add dst-address=0.0.0.0/0 gateway=90.90.0.2

# Ustawienie DNS z pozwoleniem na odpowiadanie na żądania dla urządzeń:
/ip dns set servers=208.67.222.222,208.67.220.220 allow-remote-requests=yes

# Ustawienie adresacji dla LAN (np. ether2):
/ip address add address=10.90.0.1/24 interface=ether2
```
*Uwaga: Serwer DHCP domyślnie jest wyłączony, więc nie trzeba go konfigurować, ale jeśli istniałby z domyślnej konfiguracji, usuwamy go poleceniem `/ip dhcp-server remove [find]`.*

**Jak odczytać zrobioną konfigurację (weryfikacja):**
*   Sprawdzenie IP: `/ip address print`
*   Sprawdzenie bramy (routingu): `/ip route print`
*   Sprawdzenie serwerów DNS: `/ip dns print`

---

## 5. Skonfiguruj przełącznik
Konfiguracja na urządzeniu **MikroTik**.

**Wymagania:**
*   IP: `10.80.0.3/24`
*   Brama: `10.80.0.2`

**Rozwiązanie w MikroTik (WinBox / CLI):**

```routeros
# Jeżeli switch to urządzenie działające pod kontrolą RouterOS, przypisujemy IP zazwyczaj do wbudowanego interfejsu bridge (np. bridge1), w którym spięte są porty.
/ip address add address=10.80.0.3/24 interface=bridge1

# Ustawienie bramy domyślnej (np. z zarządzania):
/ip route add dst-address=0.0.0.0/0 gateway=10.80.0.2
```

**Jak odczytać zrobioną konfigurację (weryfikacja):**
*   Sprawdzenie przypisanego adresu IP: `/ip address print`
*   Sprawdzenie trasy domyślnej: `/ip route print`

---

## 6. Połączenie urządzeń

Zgodnie ze schematem z arkusza (oraz wynikającymi z sieci adresacjami):
1. Kabel łączy interfejs WAN Rutera z zewnętrznym punktem dostępowym stanowiska egzaminacyjnego (lub stacją imitującą sieć Internet).
2. Interfejs LAN Rutera jest fizycznie połączony z pierwszą kartą sieciową Serwera.
3. Druga karta sieciowa Serwera jest podłączona do dowolnego portu dostępowego Przełącznika.
4. Stacja Robocza (interfejs oznaczony LAN_S) jest podłączona do dowolnego portu dostępowego Przełącznika.

---

## 7. Skonfiguruj serwer & 8. Napisz skrypt powłoki konta.sh

Zgodnie z wymaganiami z arkusza dla zadania 7 (Konfiguracja 2 interfejsów i DHCP) i 8 (skrypt), oraz regułą D z pliku `README.md`, poniżej przygotowano warianty rozwiązań.

### Wariant D1 - System Windows Server 2022

**Zadanie 7 - Konfiguracja Sieci:**
1. Otwórz `ncpa.cpl` (Połączenia sieciowe). Zmień nazwy interfejsów, np. "DO_RUTERA" i "DO_PRZELACZNIKA" dla lepszej identyfikacji.
2. Zmień właściwości IPv4 interfejsu "DO_RUTERA":
   - IP: `90.90.0.3`
   - Maska: `255.255.255.248` (/29)
   - Brama: `90.90.0.1`
   - DNS: Puste.
3. Zmień właściwości IPv4 interfejsu "DO_PRZELACZNIKA":
   - IP: `10.80.0.2`
   - Maska: `255.255.255.0` (/24)
   - Brama: Puste.
   - DNS: `10.80.0.2`

*Weryfikacja:* Otwórz PowerShell i wpisz `ipconfig /all` lub `Get-NetIPAddress`.

**Zadanie 7 - Usługa DHCP:**
1. Otwórz `Server Manager` -> `Add Roles and Features`. Zainstaluj `DHCP Server`.
2. Otwórz menedżer DHCP (`dhcpmgmt.msc`).
3. W drzewie serwera, kliknij prawym przyciskiem na `IPv4` i wybierz `New Scope`.
4. Skonfiguruj pulę:
   - Adres początkowy: `10.80.0.10`, Końcowy: `10.80.0.30`
   - Subnet mask: `255.255.255.0` (/24).
   - Opcje DHCP (Configure DHCP Options): Brama: `10.80.0.2`, DNS: `10.80.0.2`, Domena: `egzamin.edu`.
5. Ustawienia dzierżawy: We właściwościach Scope włącz modyfikację dzierżawy (Lease duration) domyślnie na 1 godzinę. Maksymalny czas najczęściej w Windowsie wymaga modyfikacji rejestru lub skryptowania.

**Zadanie 8 - Skrypt:**
1. W Powershell Utwórz katalog i wejdź do niego: `mkdir C:\Users\Administrator\skrypty_admina ; cd C:\Users\Administrator\skrypty_admina`.
2. Ze względu na brak wbudowanego rozdzielenia uprawnień dla poszczególnych "grup" jak w Linuksie (Tabela 3), na systemie Windows używamy po prostu zakładki `Zabezpieczenia` pliku lub komend ICACLS (np. zakaz modyfikacji dla Innych).
3. Tworzymy skrypt `konta.ps1`:
```powershell
param (
    [Parameter(Mandatory=$true)][string]$PrefixUsera,
    [Parameter(Mandatory=$true)][string]$PrefixKatalogu
)

for ($i=1; $i -le 5; $i++) {
    $userName = "$PrefixUsera$i"
    $dirName = "$PrefixKatalogu$i"
    $homePath = "C:\home\$dirName"

    # Tworzenie katalogu
    New-Item -ItemType Directory -Path $homePath -Force

    # Tworzenie użytkownika
    New-LocalUser -Name $userName -NoPassword
}
```
Uruchomienie: `.\konta.ps1 -PrefixUsera "uczen" -PrefixKatalogu "3TI"`

---

### Wariant D2 - System Linux Ubuntu Server

**Zadanie 7 - Konfiguracja Sieci:**
Plik konfiguracyjny (Netplan) zazwyczaj znajduje się w `/etc/netplan/` (np. `00-installer-config.yaml`). Wpisujemy (przykładowo na `enp0s3` i `enp0s8`):
```yaml
network:
  version: 2
  ethernets:
    enp0s3: # do rutera
      addresses:
        - 90.90.0.3/29
      routes:
        - to: default
          via: 90.90.0.1
      nameservers:
        addresses: []
    enp0s8: # do przełącznika
      addresses:
        - 10.80.0.2/24
      nameservers:
        addresses: [10.80.0.2]
```
Akceptujemy poprzez komendę `sudo netplan apply`.
*Weryfikacja:* wpisz polecenie `ip a` oraz `ip route`.

**Zadanie 7 - Usługa DHCP:**
1. Zainstaluj usługę: `sudo apt-get install isc-dhcp-server`.
2. W pliku `/etc/default/isc-dhcp-server` ustaw interfejs przypisany do przełącznika, np. `INTERFACESv4="enp0s8"`.
3. Edytuj plik `/etc/dhcp/dhcpd.conf`:
```text
authoritative;
default-lease-time 3600; # 1 godzina
max-lease-time 10800; # 3 godziny
option domain-name "egzamin.edu";
option domain-name-servers 10.80.0.2;

subnet 10.80.0.0 netmask 255.255.255.0 {
    range 10.80.0.10 10.80.0.30;
    option routers 10.80.0.2;
    option broadcast-address 10.80.0.255;
}
```
4. Zrestartuj usługę: `sudo systemctl restart isc-dhcp-server`. Sprawdź status `systemctl status isc-dhcp-server`.

**Zadanie 8 - Skrypt powłoki konta.sh:**
1. Przejdź do katalogu domowego, stwórz folder: `cd ~ ; mkdir skrypty_admina ; cd skrypty_admina`.
2. Zbuduj skrypt w pliku `konta.sh`:
```bash
#!/bin/bash
# $1 - pierwszy parametr
# $2 - drugi parametr

for i in {1..5}; do
    USERNAME="$1$i"
    HOMEDIR="/home/$2$i"

    # Tworzenie katalogu dla bezpieczeństwa
    mkdir -p "$HOMEDIR"

    # Tworzenie usera
    useradd -d "$HOMEDIR" -m "$USERNAME"
done
```

3. Nadaj uprawnienia używając notacji liczbowej, gdzie właściciel (7) to rwx, grupa (1) to --x, inni brak.
   - `chmod 710 konta.sh`
4. Wykonaj skrypt (jako root, z uwagi na komendę useradd):
   - `./konta.sh uczen 3TI`

---

## 9. Skonfiguruj system na stacji roboczej

Poniższe instrukcje odpowiadają na wariant C1 (Windows) oraz C2 (Ubuntu). Adres IP stacji roboczej pozyskany po LAN automatycznie (Klient DHCP) od serwera DHCP połączy ją z siecią `10.80.0.0/24`.

### Wariant C1 - System Windows

1. **Konfiguracja klienta DHCP (LAN_S):**
   - Wejdź w Ustawienia Sieci (ncpa.cpl).
   - Wybierz interfejs przewodowy, kliknij prawym przyciskiem i wybierz *Zmień nazwę*, wpisz `LAN_S`.
   - Kliknij *Właściwości* -> *Protokół internetowy w wersji 4 (TCP/IPv4)* i upewnij się, że wybrane są opcje "Uzyskaj adres IP automatycznie" i "Uzyskaj adres serwera DNS automatycznie".
2. **Podsystem Windows dla systemu Linux (WSL):**
   - Otwórz *Panel sterowania* -> *Programy* -> *Włącz lub wyłącz funkcje systemu Windows*.
   - Zaznacz na liście pole **Podsystem Windows dla systemu Linux** (Windows Subsystem for Linux) i zrestartuj komputer.
3. **Konto asystent:**
   - Otwórz Wiersz poleceń jako Administrator i wpisz: `net user asystent /add`. Konto będzie standardowe.
4. **Ograniczenia na dysku (Quota 8 GB, limit 7 GB):**
   - W eksploratorze plików kliknij prawym na dysk (np. C:) i wybierz `Właściwości`.
   - Przejdź na zakładkę `Przydział` (Quota) -> `Pokaż ustawienia przydziału` (Show Quota Settings).
   - Zaznacz "Włącz zarządzanie przydziałami" i kliknij na dole `Wpisy przydziału` (Quota Entries).
   - Z górnego menu wybierz "Nowy wpis przydziału" -> znajdź użytkownika `asystent`.
   - Zaznacz opcję ograniczenia dysku: Limit: `8 GB`, Poziom ostrzeżeń: `7 GB`.
5. **Zablokowanie możliwości zmiany tła pulpitu:**
   - Otwórz Edytor lokalnych zasad grupy (`gpedit.msc`).
   - Przejdź do: *Konfiguracja użytkownika* -> *Szablony administracyjne* -> *Panel sterowania* -> *Personalizacja*.
   - Znajdź i włącz (Enabled) zasadę `Zapobiegaj zmianie tła pulpitu` (Prevent changing desktop background).
6. **Wskaźnik kursora tekstu (zielony):**
   - Otwórz **Ustawienia** -> **Ułatwienia dostępu** (Accessibility) -> **Wskaźnik kursora tekstu** (Text cursor indicator).
   - Przesuń suwak na "Włączone" i w palecie kolorów wybierz/zdefiniuj kolor zielony.

### Wariant C2 - System Linux Ubuntu

1. **Konfiguracja klienta DHCP (LAN_S):**
   - Otwórz ustawienia sieciowe w GUI (ikona sieci w prawym rogu -> Ustawienia przewodowe).
   - Nazwij profil IPv4 połączenia przewodowego `LAN_S` oraz w zakładce `IPv4` wybierz metodę `Automatycznie (DHCP)`. W oknie terminala odpowiednikiem edycji byłaby modyfikacja pliku Netplan lub `nmcli con mod "Wired connection 1" connection.id "LAN_S" ipv4.method auto`.
2. **Podsystem Windows (WSL) – uwaga:**
   - Środowisko to nie istnieje w systemie Linux (jest to cecha specyficzna systemu Microsoft). Zadanie to pomijamy na Ubuntu.
3. **Konto asystent:**
   - Wpisz w terminal: `sudo useradd -m asystent` lub `sudo adduser asystent`.
4. **Ograniczenia Quota:**
   - Zainstaluj pakiet quota: `sudo apt install quota`.
   - Edytuj `/etc/fstab` dopisując `usrquota` na partycji np. na korzeniu `/`. (wymaga remountu: `sudo mount -o remount /`).
   - Wpisz komendę konfigurującą i wygeneruj pliki: `sudo quotacheck -cum /` a potem włącz ograniczenia `sudo quotaon -v /`.
   - Nadaj limit użytkownikowi asystent wpisując `sudo edquota -u asystent`. W otwartym edytorze ustaw limit miękki (soft) na `7340032` KB (7 GB) i twardy (hard) na `8388608` KB (8 GB).
5. **Blokada Tła Pulpitu:**
   - Użyj narzędzia do zarządzania środowiskiem graficznym `dconf` np: `dconf write /org/gnome/desktop/background/picture-uri "'file:///path/to/image'"` a następnie zablokuj ustawienia w `/etc/dconf/db/local.d/locks/background`. Alternatywnie, z GUI niekiedy brak prostej natywnej reguły.
6. **Wskaźnik kursora tekstu (zielony):**
   - Otwórz okno terminala (np. GNOME Terminal).
   - W `Preferencjach` profilu przejdź na `Kolory`, odznacz używanie z motywu i zmień opcję koloru `Kursora` ręcznie na zielony w palecie barw.

---

## 10. Wykonaj testy komunikacji serwera i stacji

Na serwerze (np. poprzez terminal powershell lub bash):

1. **Wykonaj test (z serwera):**
   - ping do stacji roboczej: `ping <IP_STACJI>` (np. ping 10.80.0.10)
   - ping do przełącznika: `ping 10.80.0.3`
   - ping do interfejsu WAN rutera: `ping 90.90.0.1`

2. **Wyświetlenie adresu IP interfejsu stacji roboczej (podczas testów):**
   - Na Windowsie (Stacja C1): Wpisz polecenie `ipconfig` lub wejdź we właściwości sieci z paska zadań.
   - Na Ubuntu (Stacja C2): Wpisz polecenie `ip addr show` na terminalu (IP musi figurować w podsieci 10.80.0.x).

3. **Zmiana waporach dla testu komunikacji (ICMP):**
   - Jeśli *stacja robocza (Windows)* nie odpowiada na ping serwera (domyślny przypadek w środowisku Microsoft), otwórz **Zaporę Windows Defender (wf.msc)**. Przejdź do `Reguł przychodzących` i włącz odpowiednią opcję "Udostępnianie plików i drukarek (żądanie echa - ruch przychodzący ICMPv4)" (File and Printer Sharing (Echo Request - ICMPv4-In)).
   - Dla *stacji roboczej (Ubuntu)* domyślnie pakiety ping są przepuszczane. Ewentualnie przy włączonym UFW: `sudo ufw allow icmp`.


---

## Tabele egzaminacyjne

Poniżej przygotowano wzory z prawidłowymi sposobami wypełnienia po przeprowadzeniu weryfikacji sprzętowej w poszczególnych systemach (patrz **Zadanie 2** dla sposobu odczytu parametrów).

### Tabela 2. Parametry systemu i podzespołów stacji roboczej oraz ocena

| System operacyjny       |  Wpisz zrzut parametru np. `Windows 10 Pro` |
|------------------------|------------------------------------------|
| Nazwa i typ            |  Wpisz zrzut parametru np. `x64` |
| Nazwa i oznaczenie     |  Wpisz dane odczytane w zad 2 np. `Intel Core i5-10400` |
| Procesor (Taktowanie)  |  Wpisz taktowanie np. `2.9 GHz` |
| Karta graficzna (Producent)| Wpisz producenta np. `NVIDIA` |
| Wirtualizacja (Włączona)| **TAK** *(Zgodnie z weryfikacją na Menedżerze Zadań / lscpu)* |
| Pamięć RAM (Rozmiar)   |  Wpisz np. `8 GB` lub `16 GB` |
| Pamięć RAM (Rodzaj)    |  Wpisz np. `DDR4` |
| Dysk twardy (Pojemność)|  Wpisz np. `500 GB` |
| **Ocena**              | W oparciu o Tabelę 1 (Win10, i3/Ryz3, 8GB RAM, SSD, Virt: ON) napisz: **„Zgodnie z przeprowadzonymi testami (lub ich brakiem dla SSD) stacja pozwala/nie pozwala na korzystanie z programu, ponieważ procesor (...) spełnia wymogi, podobnie wielkość pamięci RAM (...).”** (Odnieś się do odczytanych wyżej wyników). |

---

### Tabela 3. Polecenia Linux

Zgodnie z wykonanym zadaniem nr 8 w systemie Linux (Serwer - D2):

| Działanie systemowe | Użyte polecenie |
|---------------------|-----------------|
| Polecenie nadające prawa za pomocą notacji liczbowej do pliku zawierającego skrypt | `chmod 710 konta.sh` |
| Polecenie uruchamiające skrypt z parametrami: uczen oraz 3TI | `./konta.sh uczen 3TI` |
