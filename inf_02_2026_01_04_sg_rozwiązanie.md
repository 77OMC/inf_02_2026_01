# Rozwiązanie egzaminu INF.02-01-26.04-SG

**Wersja arkusza:** SG
**Sesja:** Czerwiec 2025 (Zgodnie z poleceniem)

---

## 1. Wykonaj na stacji roboczej montaż zapasowej karty sieciowej

**Opis wykonania:**
Należy przestrzegać podstawowych zasad BHP przy pracy z podzespołami:
1. Odłącz stację roboczą od sieci zasilającej (wyjmij kabel zasilający z zasilacza).
2. Rozładuj ładunki elektrostatyczne ze swojego ciała (np. używając opaski antystatycznej na nadgarstek uziemionej do obudowy).
3. Otwórz obudowę komputera. Jeżeli karta sieciowa jest wbudowana na płycie (zintegrowana) uruchom najpierw komputer i wyłącz ją z poziomu systemu (Menedżer Urządzeń) lub poprzez menu BIOS/UEFI.
4. Zdemontuj zainstalowaną przewodową kartę rozszerzeń, odkręcając śrubę ze śledzia obudowy i wyciągając ją z portu PCI/PCIe.
5. Zainstaluj w odpowiednim gnieździe na Płycie Głównej zapasową kartę sieciową znajdującą się na wyposażeniu.
6. Przykręć śledź nowej karty, aby zamocować ją sztywno w obudowie i zabezpieczyć sprzęt.
7. Zgłoś Przewodniczącemu ZN przez podniesienie ręki wykonanie montażu.
8. Po uzyskaniu zgody zamknij obudowę, podepnij sprzęt do prądu i uruchom stację roboczą.

---
## 2. Przeprowadź identyfikację podzespołów komputera

Zadanie w arkuszu nakazuje diagnozę przez stację w systemie Windows. Poniżej omówiono to zadanie oraz przygotowano też rozwiązanie dla wariantu C2 podpartego wymogami pliku README.

### Wariant C1 - System Windows (Domyślny z arkusza)
Zainstaluj narzędzie CPU-Z znajdujące się na wskazanym nośniku, przechodząc prosty kreator instalacyjny. Następnie zlokalizuj wymagane do Tabeli 1 dane:

1. **Producent i model płyty głównej:** Uruchom program CPU-Z i otwórz zakładkę **Mainboard**. Parametry te ukrywają się pod pozycjami *Manufacturer* oraz *Model*. Wykonaj zrzut.
2. **Liczba rdzeni procesora:** W CPU-Z w głównym oknie (zakładka **CPU**), w prawym dolnym rogu jest pole *Cores*. Odczytaj stamtąd liczbę fizycznych rdzeni (zrzut z zakładką CPU).
3. **Typ gniazda procesora:** Tamże (w zakładce CPU), odczytaj parametr pod pozycją *Package* (np. Socket AM4 lub LGA 1151).
4. **Taktowanie podstawowe procesora:** Tamże (zakładka CPU) lub wbudowanym `Menedżerze Zadań (Wydajność)`. W CPU-Z sprawdź parametr *Core Speed* z tabeli *Clocks* lub informację w *Specification*.
5. **Technologia wykonania procesora:** Tamże (zakładka CPU), pozycja określana jako *Technology* (np. 14 nm).
6. **Wykorzystanie procesora w % (chwilowe):** Odpal narzędzie **Menedżer zadań** (`Ctrl+Shift+Esc`), wejdź na kartę **Wydajność** (Performance) i wybierz Procesor. Wykonaj zrzut ekranu widocznego wykresu i parametru *Wykorzystanie* (Utilization).

Wszystkie zrzuty zapisz we wskazanym folderze `PARAMETRY` na dysku pendrive. Parametry przepisz do Tabeli 1.

### Wariant C2 - System Linux Ubuntu Desktop
Oto te same operacje w systemie Linux (np. używając wbudowanych programów, ponieważ CPU-Z nie występuje jako stabilna paczka graficzna w domyślnych repach linuksa)

1. **Płyta Główna i typ gniazda CPU:** Odpal terminal i wpisz `sudo dmidecode -t baseboard` lub ew. `sudo lshw -class board` do sprawdzenia producenta i płyty głównej. Z kolei polecenie `sudo dmidecode -t processor | grep Upgrade` wskaże na złącze np Socket AM4.
2. **Liczba rdzeni, Technologia i Taktowanie:** Użyj terminalowego narzędzia do sprawdzenia topologii procesora `lscpu`. Dowiesz się z niego ilości rdzeni (*Core(s) per socket*), z odnośników można wylistować w internecie litografie w *nanometrach*, a taktowanie to np. `Model name:` lub konkretnie z parametru `CPU MHz`.
3. **Chwilowe użycie w %:** Wpisz i odpal w terminalu popularne narzędzie `top` lub graficzny "Monitor Systemu". Zrób zrzut ekranu całego pulpitu naciskając przycisk PrintScreen.

---
## 3. Skonfiguruj ruter
Zgodnie z wytyczną `B` (`README.md`), konfigurację oparto o urządzenia **MikroTik**.

**Wymagania:**
*   Interfejs WAN IP: `88.88.88.2/30`
*   Brama domyślna WAN: `88.88.88.1`
*   Serwery DNS: `6.6.6.6` oraz `7.7.7.7`
*   Interfejs LAN IP: `172.16.10.10/24`
*   DHCP wyłączony (lub nieskonfigurowany).

**Rozwiązanie w MikroTik (Terminal / WinBox):**
```routeros
# Ewentualny reset do ustawień "pustych" na start
/system reset-configuration no-defaults=yes skip-backup=yes

# Ustawienie adresacji dla WAN (przyjmując że połączony to ether1):
/ip address add address=88.88.88.2/30 interface=ether1

# Ustawienie bramy domyślnej rutera wychodzącej na zewnątrz:
/ip route add dst-address=0.0.0.0/0 gateway=88.88.88.1

# Ustawienie serwerów DNS na ruterze
/ip dns set servers=6.6.6.6,7.7.7.7 allow-remote-requests=yes

# Ustawienie adresacji dla interfejsu LAN
/ip address add address=172.16.10.10/24 interface=ether2
```

**Jak odczytać skonfigurowane ustawienia (weryfikacja w CLI MT):**
*   Adresacja: `/ip address print`
*   Trasy i bramy: `/ip route print`
*   Opcje DNS: `/ip dns print`

---

## 4. Skonfiguruj przełącznik
Zgodnie z wytyczną `B` (`README.md`), konfigurację oparto o urządzenia **MikroTik**. Poniżej zakładamy switch pracujący na bazie RouterOS.

**Wymagania:**
*   Adres IP: `10.0.10.5/24`
*   Brama: adres IP interfejsu LAN W serwera (`10.0.10.1`)

**Rozwiązanie w MikroTik (WinBox / CLI Terminal):**
```routeros
# Zakładając że porty są spięte w lokalnym bridge'u np. bridge1
/ip address add address=10.0.10.5/24 interface=bridge1

# Ustawienie bramy domyślnej dla zarządzania przełącznika na interfejs wewnętrzny Serwera LAN W
/ip route add dst-address=0.0.0.0/0 gateway=10.0.10.1
```

**Jak odczytać zrobioną konfigurację (weryfikacja):**
*   Sprawdzenie adresu IP: `/ip address print`
*   Sprawdzenie trasy domyślnej: `/ip route print`

---

## 5. Połączenie urządzeń

Na bazie schematu podłączenia sieci budujemy następującą strukturę:
1. Pomiędzy serwerem, a stacją roboczą, pośredniczy przełącznik. Wszelkie końcówki na tych urządzeniach oznaczone na serwerze i stacji jako LAN W, LAN K mają wtyczkę RJ-45 w przełączniku.
2. Z drugiej karty serwera - interfejs LAN Z podłączony jest bezpośrednio kablem prostym do interfejsu LAN wewnątrz Rutera.
3. Ruter poprzez interfejs WAN jest wyciągnięty na zewnątrz za pomocą kabla miedzianego i połączony prawdopodobnie z E-X, by móc odsyłać pakiety ICMP do innej strefy.
4. Całość sprzętu zostaje uruchomiona podpięta pod stałe zasilanie sieciowe 230V.

---

## 6. Na stacji roboczej skonfiguruj system (Interfejs, nazwa, katalog i pulpit)

Zadanie narzuca z góry system Linux na stacji, niemniej na potrzeby dokumentacji dla wytycznych C z `README.md`, zostają rozpisane oba warianty na równi, z naciskiem na C2 z arkusza 04.

### Wariant C2 - System Linux Ubuntu Desktop (Domyślny)

1. **Konfiguracja adresacji LAN K (z konsoli lub graficznie):**
   - Otwórz Ustawienia -> Sieć lub w terminalu wyedytuj połączenie `nmcli connection modify "Połączenie przewodowe 1" connection.id "LAN K" ipv4.method manual ipv4.addresses 10.0.10.11/24 ipv4.gateway 10.0.10.1 ipv4.dns 10.0.10.1`.
   - Zrestartuj profil sieciowy aby załapał statyczne adresy: `nmcli con down "LAN K" && nmcli con up "LAN K"`.
2. **Zmiana nazwy komputera:**
   - W środowisku Linuksowym wpisz w terminal polecenie: `sudo hostnamectl set-hostname stanowiskoXX` (gdzie XX to wyznaczony numer w sali komputerowej np stanowisko01). Będzie to widoczne natychmiast, wymaga przelogowania by zmienić prompt bash'a.
3. **Odczyt adresu fizycznego (Tabela 3):**
   - Aby sprawdzić adres MAC używa się komendy: `ip link show` (gdzie wiersz `link/ether` definiuje ten adres).
   - *Alternatywa to polecenie `ifconfig` w paczce net-tools podające `ether`.* Wpisz te dane w Tabeli 3 w arkuszu.
4. **Katalog z uprawnieniami w katalogu domowym:**
   - Przejdź do folderu domowego użytkownika administrator (zazwyczaj `/home/administrator`): `cd ~`
   - Utwórz katalog i nadaj mu stosowne uprawnienia z notacji liczbowej, co można zrealizować podanym niżej złączonym poleceniem:
     ```bash
     mkdir ELEKTRONIKA && chmod 741 ELEKTRONIKA
     ```
   - (7 - pełne rwx dla Właściciel, 4 - odczyt r-- dla Grupa, 1 - wykonanie --x dla Innych). Odczytaj uprawnienia dla egzaminatora komendą `ls -ld ELEKTRONIKA`.
5. **Ustawienie Tapety Pulpitu:**
   - Skopiuj najpierw zdjęcie na dysk komputera ze sprzętu egzaminacyjnego: np. `cp /media/usb/PLIKI/water.jpg ~/Obrazy/water.jpg`
   - Następnie kliknij na nim PPM "Ustaw jako tło pulpitu" lub poprzez linię poleceń wpisz do `dconf`/`gsettings`: `gsettings set org.gnome.desktop.background picture-uri 'file:///home/administrator/Obrazy/water.jpg'`.

### Wariant C1 - System Windows

Jeśli stanowiskiem jest system operacyjny Windows, postępuj zgodnie z:
1. Skonfiguruj sieć wpisując `ncpa.cpl`. Zmień nazwę karty na `LAN K`. Zmień `Właściwości` w zakładce `Protokół internetowy IPv4` - Zaznacz na sztywno IP: `10.0.10.11`, Maska: `255.255.255.0`, Brama i preferowany DNS to: `10.0.10.1`.
2. Wpisz `sysdm.cpl` by zmienić `Nazwę Komputera` (Kliknij Zmień, a w pole Nazwa wklep `stanowiskoXX`). Skutkuje to zrestartowaniem systemu komputera.
3. Adres Fizyczny MAC znajdziesz wydając polecenie `getmac /v` na wierszu CMD, co stanowi dowód wpisany do Tabeli 3 obok wybranego polecenia.
4. Utworzenie katalogu to po prostu `mkdir C:\Users\Administrator\ELEKTRONIKA`. Uprawnienia nadaje się poleceniem ICACLS, ewentualnie wejdź we Właściwości folderu na zakładkę *Zabezpieczenia* i *Zaawansowane*, gdzie Wyłączasz dziedziczenie. Pozostaw Właściciela z "Pełną kontrolą", Administratorom "Pełną Kontrolę", a jakiejś wybranej Grupie zablokuj wszystko i pozwól tylko na odczyt zawartości foldera, itd.
5. Ustawienie tapety polega na zgraniu z dysku USB opisanego obrazka i wybrania na nim z menu kontekstowego `Ustaw jako tło pulpitu`.

---
## 7. Skonfiguruj serwer z systemem operacyjnym Windows

Zadanie narzuca z góry system Windows dla serwera, niemniej na potrzeby dokumentacji dla wytycznych README.md z podziałem C/D zostają stworzone opcje (D1 - Windows, D2 - Ubuntu Server).

### Wariant D1 - System Windows Server (Domyślny)

1. **Konfiguracja Adresacji Serwera (ncpa.cpl):**
   - Połączona z ruterem: zmień na `LAN Z`. W protokole IPv4 ustaw stały adres `172.16.10.1`, maskę `255.255.255.0`, bramę `88.88.88.2` (lub inną wg faktycznego schematu, najprawdopodobniej to sam ruter jest w LANie 172.16.10.10 i służy jako brama wewn.), oraz adres DNS na localhost `127.0.0.1`.
   - Połączona z przełącznikiem: zmień na `LAN W`. Ustaw stały adres `10.0.10.1`, maskę `255.255.255.0`, bramę zostaw PUSTĄ, a DNS na `127.0.0.1`.
2. **Promocja Serwera do Kontrolera Domeny (AD DS):**
   - W Menedżerze Serwera, z głównej tablicy kliknij "Dodaj role i funkcje". Zaznacz rolę `Usługi domenowe Active Directory` i zainstaluj z ustawieniami standardowymi.
   - Po instalacji w lewym górnym rogu pod żółtą flagą (trójkątem) wybierz **Promuj ten serwer do roli kontrolera domeny**.
   - W okienku wdrażania zaznacz: *Dodaj nowy las* i podaj nazwę domeny w roocie np. `firma05.local`. Wciśnij Dalej.
   - Podaj opcjonalne hasło do logowania z trybu przywracania usług (DSRM): `ZAQ!2wsx`. Zignoruj delegacje DNS z panelu, oraz klikaj instaluj zgadzając się ze wszystkimi ostrzeżeniami o zabezpieczeniach.
   - Komputer ulegnie ponownemu uruchomieniu. Ewentualnie podmień hasło standardowego Administratora na systemowe wciskając `Ctrl+Alt+Del` (w AD lub Lokalnie) i wybierając opcję Zmień Hasło na `zaq1@WSX`.
3. **Zasady grupy (GPO) "Polityka":**
   - Wciśnij na klawiaturze `Win+R` i otwórz `gpmc.msc` (Zarządzanie zasadami grupy).
   - Prawym przyciskiem w drzewie zarządzania domeną `firmaXX.local` w lewym oknie wybierz `Utwórz obiekt zasad grupy GPO w tej domenie`.
   - Wpisz nazwę reguły na `Polityka`. Kliknij edytuj, żeby zacząć ją konfigurować.
   - Dojdź do drzewa `Konfiguracja komputera -> Zasady -> Ustawienia systemu Windows -> Ustawienia zabezpieczeń -> Zasady konta -> Zasady haseł`.
   - Zmień właściwość *Hasło musi spełniać wymagania co do złożoności* na włączoną, oraz *Minimalna długość hasła* na `8 znaków`. Zatwierdź GPO.
   - Na wierszu powłoki wpisz polecenie `gpupdate /force` aby zaktualizować na domenie nowe wartości obostrzeń natychmiastowo.
4. **Jednostka organizacyjna i Użytkownicy:**
   - Otwórz aplikację *Użytkownicy i komputery usługi Active Directory* (`dsa.msc`).
   - Prawym klawiszem na domenę w folderze -> `Nowy -> Jednostka Organizacyjna (Organizational Unit)`. Wpisz nazwę `BIURO`.
   - Kliknij na folder BIURO i znów `Nowy -> Użytkownik`.
   - Zrób usera 1: Imię: Krystyna, Nazwisko: Nowak, Logowanie: `sekretarka`. Nadaj jej dowolne hasło łapiące złożoność (np. `Kow@lsk!123` lub używając zasady DSRM) i zapisz wygenerowane/użyte w Tabeli 2 w oznaczonym wierszu.
   - Zrób usera 2: Imię: Jan, Nazwisko: Kowalski, Logowanie: `elektronik`. Dodaj kolejne hasło spełniające zbiory zasad AD i zapisz do Tabeli 2 z arkusza.

### Wariant D2 - System Linux Ubuntu Server
Tworzenie Kontrolera AD na Linux odbywa się bez GUI.
1. Karty LAN pod interfejsy `netplan` należy zapisać w /etc/netplan pod maski /24 analogicznie z adresami `172.16.10.1` z DNS `127.0.0.1` dla karty zewnętrznej i dla drugiej `10.0.10.1` ze stałą siecią.
2. Usługa Active Directory jest obsługiwana w pakiecie SAMBA jako *Samba4 AD DC*. Należy pobrać paczkę serwisu, oraz stworzyć domenę wykorzystując narzedzie CLI:
   `samba-tool domain provision --use-rfc2307 --realm=firmaXX.local --domain=FIRMAXX --server-role=dc --dns-backend=SAMBA_INTERNAL --adminpass=zaq1@WSX`
3. Konfiguracja polityk dla GPO za pomocą `samba-tool`:
   `samba-tool domain passwordsettings set --complexity=on` oraz `samba-tool domain passwordsettings set --min-pwd-length=8`.
4. Tworzenie użytkowników wg Tabeli 2:
   - `samba-tool user create sekretarka Kow@lsk!123 --given-name="Krystyna" --surname="Nowak"`
   - `samba-tool user create elektronik Kow@lsk!456 --given-name="Jan" --surname="Kowalski"`
   (Na żądanie wpisz odpowiednio wymyślone powyższe hasła w tabele w arkuszu).


---

## 8. Utwórz plik wsadowy "powitanie"

Zgodnie z poleceniem i narzuconym plikiem `.bat`, stwarzamy skrypt w głównym katalogu `C:\` w Windows.

### Wariant D1 (Windows CMD)
Skrypt w środowisku Batch (cmd):
Utwórz i wklej zawartość do pliku `C:\powitanie.bat`:
```cmd
@echo off
cls
set /p imie="Podaj imie: "
echo Witaj %imie%
pause
```
Objaśnienia do wymagań:
* `@echo off` - wyłącza wyświetlanie treści wpisywanych poleceń skryptu.
* `cls` - kasuje terminal (wyczyszczenie ekranu).
* `set /p` pobiera wpisywany w wierszu wynik parametru jako zmienną `imie`.
* `echo Witaj %imie%` - zwraca tekst i nazwę.
* `pause` - wstrzymuje terminal by okno od razu się nie wyłączyło zatrzymując tym samym skrypt. Użytkownik musi kliknąć jakikolwiek przycisk klawiatury by zwolnić blokadę.

### Wariant D2 (Bash Ubuntu)
Gdyby była to forma skryptu powłoki `.sh` plik na ścieżce np. `/powitanie.sh` (pamiętaj z `chmod +x`):
```bash
#!/bin/bash
clear
read -p "Podaj imie: " imie
echo "Witaj $imie"
read -s -n 1 -p "Nacisnij dowolny klawisz, aby zakonczyc skrypt..."
```
Oczekuje on i pobiera z `read` informację oraz nie zamyka sie (blokuje wątek poleceniem na 1 char) dopóki się go jawnie nie wymusi interakcją z systemem.

---
## 9. Testy Komunikacji i tabele egzaminacyjne

Otwórz linię poleceń na serwerze i sprawdź stabilność podłączenia za pomocą echa ICMP puszczając pakiety za pomocą narzędzia Ping. (Możesz potrzebować zezwolić zaporze na odbieranie ping na stacji C1 regułami firewall dla pingu v4 - chociaż w wariancie zadania 04 jest C2, Linux standardowo potrafi poprawnie puszczać połączony ping w sieci wew.).

Wykonaj z poziomu serwera:
1. Ping do stacji: `ping 10.0.10.11`
2. Ping do przełącznika: `ping 10.0.10.5`
3. Ping do LAN rutera: `ping 172.16.10.10`

### Tabela 1. Parametry podzespołów stacji roboczej
Według identyfikacji podanej w punkcie 2, wprowadź przykładowe rzeczywiste zrzuty lub logi do Tabeli 1 z dokumentu. Np.:
| Parametr | Pobrana wielkość |
|----------|-----------------|
| Producent i model płyty głównej | `ASUSTeK COMPUTER INC. PRIME B450-PLUS` |
| Liczba rdzeni procesora | `6` |
| Typ gniazda procesora | `Socket AM4 (1331)` |
| Taktowanie podstawowe procesora | `3600 MHz` / `3.60 GHz` |
| Technologia wykonania procesora | `7 nm` |
| Wykorzystanie procesora w % (chwilowe) | `2%` |

---

### Tabela 2. Konta domenowe
Według utworzonych kont (Zadanie 7), należy przypisać do tabeli wymyślone samodzielnie zgodnie z długością i GPO hasła. Np.:
| Pełna nazwa | Nazwa logowania | Hasło |
|-------------|-----------------|-------|
| Krystyna Nowak | `sekretarka` | `TrudneH@slo1` |
| Jan Kowalski | `elektronik` | `MocneH@slo2` |

---

### Tabela 3. Adres fizyczny karty sieciowej
Według diagnozy stacji w zadaniu 6, przypisz adres z polecenia, m.in. dla Linuxowego `ip link` z parametrem "link/ether":
| Adres fizyczny karty sieciowej zainstalowanej w stacji roboczej | `1A:2B:3C:4D:5E:6F` |
|-----------------------------------------------------------------|----------------------|
| Użyte polecenie | `ip link show` (dla Linux) LUB `getmac` (dla Windows C1) |
