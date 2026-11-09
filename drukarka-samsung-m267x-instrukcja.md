# Instalacja drukarki Samsung M267x 287x na komputerze z Windows — instrukcja krok po kroku

Wersja dla człowieka przy komputerze. Nie wymaga agenta, Hermesa ani żadnego modelu LM.
Wszystko robisz myszką i klawiaturą. Czas: 10–15 minut (druk), +5 minut (skaner).

> **Uwaga o nazwie modelu:** „M267x 287x Series" to jedna rodzina sterowników obejmująca
> M2670/M2675/M2870/M2875 i podobne. Jeśli na obudowie widzisz np. „M2875ND", to jest to ta sama
> rodzina — sterownik i instrukcja pasują.

---

## 0. Co musisz mieć

| Element | Wartość |
|---|---|
| Drukarka | Samsung **M267x 287x Series** (MFP — drukuje i skanuje) |
| Adres IP w sieci | **192.168.50.17** — to jest IP tej drukarki, wpisz je w kroku 3.3 |
| Nazwa sieciowa urządzenia | `SEC001599DD4762` |
| Numer seryjny | `ZD0NBJDD30066P` |
| Adres MAC (do rezerwacji DHCP) | `00-15-99-DD-47-62` |
| Brama / router | `192.168.50.1` |
| Firmware / NIC | `V3.00.01.12` / `NIC V6.01.00` |
| Komputer docelowy | Windows 10 lub 11, konto z uprawnieniami administratora |
| Sieć | ten sam router/Wi-Fi co drukarka |

Drukarka i komputer **muszą być w tej samej sieci lokalnej**. Jeśli komputer łączy się przez VPN
albo jest w innej podsieci (np. 192.168.1.x przy drukarce 192.168.50.x), instalacja po IP nie zadziała.

> **Stan na 2026-11-09 (sprawdzone w sieci domowej):** drukarka odpowiada na `192.168.50.17`,
> ICMP 5–16 ms; otwarte porty **80** (SyncThru, HTTP 200), **9100** (druk RAW), **515** (LPR),
> **631** (IPP); port 443 zamknięty. SNMP zwraca model `Samsung M267x 287x Series`,
> nazwę `SEC001599DD4762` i S/N `ZD0NBJDD30066P`. Na komputerze `KomputerAlunia` istnieje już
> kolejka `Samsung M267x 287x Series` na porcie `Samsung_M267x_9100` (RAW, 9100), którego
> `HostName` w rejestrze = `192.168.50.17`.

---

## 1. Ustal aktualny adres IP drukarki

Adres **192.168.50.17** jest aktualny w dniu konfiguracji, ale **może się zmienić** po restarcie
routera, bo drukarka nie ma rezerwacji DHCP. Trzy sposoby sprawdzenia — wystarczy jeden:

**A. Panel drukarki (najpewniejszy).** Na urządzeniu: `Menu` → `Network` → `Network Config`
(albo `Print Network Configuration`). Wydrukuje się/pojawi się strona z sekcją `TCP/IP` i pozycją
`IP Address`.

**B. Panel administracyjny routera.** Otwórz w przeglądarce `192.168.50.1`, zaloguj się,
wejdź w listę klientów DHCP i szukaj:
- nazwy hosta `SEC001599DD4762`, albo
- adresu MAC `00-15-99-DD-47-62`.

**C. Test z komputera.** Otwórz `Wiersz polecenia` (Win+R → `cmd`) i wpisz:

```
ping 192.168.50.17
```

Odpowiedzi `Odpowiedź z 192.168.50.17: bajtów=32 czas<1ms` = adres aktualny i komputer go widzi.
`Upłynął limit czasu oczekiwania` albo `Host nieosiągalny` = adres się zmienił albo sieć nie przepuszcza.

**D. Sprawdzenie portu druku (mocniejsze niż ping).** W `cmd`:

```
powershell Test-NetConnection 192.168.50.17 -Port 9100
```

`TcpTestSucceeded : True` przy porcie 9100 = na tym adresie naprawdę stoi drukarka, a nie inny host.

> **Zrób to od razu:** poproś osobę z dostępem do routera o **rezerwację DHCP** dla MAC
> `00-15-99-DD-47-62` (docelowo `192.168.50.17`). Bez tego IP może się zmienić i drukarka
> „zniknie" z komputera — port w Windows trzyma adres IP, nie nazwę.

---

## 2. Pobierz sterownik

### 2.1 Plik drukarki (to jest obowiązkowe)

Bezpośredni link:

```
https://ftp.hp.com/pub/softlib/software13/printers/SS/SL-M2670FN/M267x_287x_Series_WIN_Printer_V3.13.13.00.01_CDV1.44.zip
```

| Kontrola | Wartość |
|---|---|
| Nazwa pliku | `M267x_287x_Series_WIN_Printer_V3.13.13.00.01_CDV1.44.zip` |
| Rozmiar | **22 049 300 bajtów** (≈21 MB) |
| MD5 | `D36F00E804CD10F30B5D7412BD117025` |

**Jak sprawdzić MD5 w Windows** (uczciwa kontrola, że plik nie jest uszkodzony):

```
certutil -hashfile "%USERPROFILE%\Downloads\M267x_287x_Series_WIN_Printer_V3.13.13.00.01_CDV1.44.zip" MD5
```

Wynik musi być identyczny z powyższym (wielkość liter nie ma znaczenia).

**Jeśli link nie działa:** wejdź na `https://support.hp.com/pl-pl/drivers`, wpisz `Samsung M267x 287x`,
wybierz system Windows 10/11 (64-bit) i pozycję **Sterownik drukarki (Printer Driver)**.
Nie pobieraj „Easy Printer Manager", jeśli nie potrzebujesz funkcji „Scan to PC" z panelu drukarki.

### 2.2 Plik skanera (tylko jeśli chcesz skanować)

```
https://ftp.hp.com/pub/softlib/software13/printers/SS/SL-M2670FN/M267x_287x_Series_WIN_Scanner_V3.31.19.04_CDV1.44.exe
```

| Kontrola | Wartość |
|---|---|
| Nazwa pliku | `M267x_287x_Series_WIN_Scanner_V3.31.19.04_CDV1.44.exe` |
| Rozmiar | **27 492 824 bajtów** (≈26 MB) |
| MD5 | `C7E121346462A145B4F71A0EDDE5CDE8` |

---

## 3. Zainstaluj drukarkę

### 3.1 Rozpakuj ZIP

Kliknij plik prawym → **Wyodrębnij wszystko** → np. do `C:\SamsungM267x`.
W środku jest m.in. katalog `SPL_PCL` z plikiem `ssa6m.inf` — to plik sterownika, może się przydać
w kroku 3.3, więc **nie usuwaj rozpakowanego katalogu**, dopóki drukarka nie działa.

### 3.2 Punkt pierwszy: kreator Samsunga

W rozpakowanym katalogu uruchom `Setup.exe` (prawym → **Uruchom jako administrator**).

1. Zaakceptuj umowę.
2. Wybierz typ połączenia **Sieć / Network** (nie USB).
3. Kreator powinien sam znaleźć drukarkę po sieci i pokazać ją jako `Samsung M267x 287x Series`
   (adres `192.168.50.17`, nazwa `SEC001599DD4762`).
4. Zaznacz ją, kliknij **Dalej** i dokończ instalację.
5. Na końcu zaznacz **wydruk strony testowej**.

Jeśli kreator znalazł i zainstalował drukarkę — przejdź do kroku 4 (weryfikacja).

### 3.3 Jeśli kreator NIE znajduje drukarki — dodaj ręcznie po adresie IP

To jest ścieżka awaryjna i zarazem najbardziej przewidywalna.

1. **Ustawienia** (Win+I) → **Bluetooth i urządzenia** → **Drukarki i skanery**
   (w Windows 10: **Urządzenia** → **Drukarki i skanery**).
2. Kliknij **Dodaj urządzenie** / **Dodaj drukarkę lub skaner**.
3. Poczekaj, aż Windows nic nie znajdzie, i kliknij
   **„Nie widzę mojej drukarki na liście"** (Win 11: **Dodaj ręcznie**).
4. Wybierz **Dodaj drukarkę przy użyciu adresu IP lub nazwy hosta** → **Dalej**.
5. Typ urządzenia: **Urządzenie TCP/IP** → **Dalej**.
6. **Nazwa hosta lub adres IP:** `192.168.50.17` → **Dalej**
   (pole „Nazwa portu" wypełni się samo, np. `IP_192.168.50.17`; na obecnej maszynie port
   nazywa się `Samsung_M267x_9100` i to też jest poprawne — liczy się `HostName = 192.168.50.17`
   i `PortNumber = 9100`, protokół **RAW**).
7. Gdy Windows zapyta o sterownik, kliknij **Dysk... / Have Disk** i wskaż plik:

```
C:\SamsungM267x\SPL_PCL\ssa6m.inf
```

8. Z listy wybierz **Samsung M267x 287x Series** → **Dalej** → **Zakończ**.
9. Windows może zapytać o zaufanie do sterownika — kliknij **Zainstaluj**.

### 3.4 Jeśli Windows twierdzi, że sterownik już jest, a drukarka nie drukuje

Wróć do kroku 3.3, ale w punkcie 7 nie wskazuj INF, tylko wybierz z listy
**Samsung M267x 287x Series** (sterownik zostanie wzięty z systemu).

---

## 4. Wydruk strony testowej i sprawdzenie

1. **Ustawienia** → **Drukarki i skanery** → kliknij `Samsung M267x 287x Series` →
   **Zarządzanie** (Win 11) → **Drukuj stronę testową**
   (Win 10: prawym na drukarce → **Właściwości drukarki** → zakładka **Ogólne** → **Drukuj stronę testową**).
2. Drukarka powinna wypluć stronę z logo Windows i informacjami o sterowniku.

**Kontrola twarda (bez patrzenia na wydruk):** na panelu drukarki sprawdź licznik stron
(`Menu` → `System Setup` → `Report`/`Usage`) — powinien wzrosnąć o 1. Jeśli wolisz cyfrowo,
wejdź w przeglądarce na `http://192.168.50.17` (SyncThru Web Service) i porównaj licznik
w sekcji informacji o urządzeniu przed i po wydruku.

**Test z aplikacji:** otwórz Notatnik, wpisz `test 1 2 3`, `Ctrl+P`, wybierz
`Samsung M267x 287x Series`, drukuj. To sprawdza całą ścieżkę, a nie tylko kolejkę.

---

## 5. Skaner (opcjonalnie)

### 5.1 Zainstaluj sterownik skanera

Uruchom `M267x_287x_Series_WIN_Scanner_V3.31.19.04_CDV1.44.exe` (prawym → **Uruchom jako administrator**).
Kreator przejdzie sam; jeśli na końcu pokaże błąd o brakującym pliku pomocy
(`MANUAL\...\start_here.htm`), **zignoruj go** — sterownik jest już zainstalowany.

### 5.2 Ważne: sterownik Samsunga działa tylko z programami 32-bitowymi

Ten sterownik skanera rejestruje się jako źródło **TWAIN 32-bit**. Programy 64-bitowe
(np. IrfanView x64, NAPS2 8.x) **go nie zobaczą** — to nie jest awaria, tylko architektura.

Sprawdzony program, który działa: **NAPS2 7.4.3 x86**.

```
https://github.com/cyanfish/naps2/releases/download/v7.4.3/naps2-7.4.3-win-x86.msi
```

| Kontrola | Wartość |
|---|---|
| Rozmiar | **12 060 712 bajtów** |
| MD5 | `498092EEFC4CE3D52AB9BA7A6DED6948` |

Instalacja: dwuklik na `.msi` → **Dalej** → **Zainstaluj**. Program pojawi się jako **NAPS2**
(bez dopisku „x64" w nazwie wersji).

### 5.3 Zeskanuj stronę

1. Połóż kartkę na szybie skanera (stroną do dołu, wyrównaną do strzałki).
2. Uruchom **NAPS2**.
3. Jeśli w lewym górnym rogu nie ma wybranej drukarki, rozwiń listę i wybierz
   **Samsung M267x 287x Series** (tytuł okna zmieni się na `NAPS2 – Samsung M267x 287x Series`).
4. Kliknij **Skanuj**. Po chwili w panelu miniaturek pojawi się strona.
5. Zapisz ją: **Zapisz PDF** albo **Zapisz obraz**.

Jeśli lista urządzeń jest pusta, sprawdź, czy masz zainstalowaną wersję **x86** (krok 5.2) —
to najczęstsza przyczyna.

### 5.4 Czego nie zainstalujesz bez kombinowania

- **Skanowanie przez WIA** (wbudowane „Windows Scan", „Faks i skanowanie Windows") — ten model
  ma w sterowniku wyłącznie powiązanie USB, więc po sieci WIA go nie zobaczy.
- **„Scan to PC" z panelu drukarki** — wymaga Easy Printer Managera, którego instalator (NSIS)
  nie instaluje się poprawnie w trybie cichym i dodatkowo ciągnie Adobe Reader. Dla skanowania
  z komputera nie jest potrzebny.

---

## 6. Rozwiązywanie problemów

| Objaw | Przyczyna | Co zrobić |
|---|---|---|
| `ping` nie odpowiada | IP się zmienił / inna podsieć / VPN | wróć do kroku 1, sprawdź panel drukarki; wyłącz VPN |
| Kreator Samsunga nie widzi drukarki | blokada rozgłaszania w sieci | użyj kroku 3.3 (instalacja po IP) |
| „Nie można znaleźć sterownika" przy porcie TCP/IP | nie wskazano INF | **Dysk...** → `C:\SamsungM267x\SPL_PCL\ssa6m.inf` |
| Wydruk idzie, ale strony są puste/krzaczki | zły język druku (PCL vs SPL) | usuń drukarkę, powtórz 3.3 z pliku `ssa6m.inf` z katalogu `SPL_PCL` |
| Drukarka „offline" po restarcie routera | brak rezerwacji DHCP | ustaw rezerwację dla MAC `00-15-99-DD-47-62`, potem popraw port na nowy IP |
| NAPS2 nie widzi skanera | wersja 64-bitowa | odinstaluj, zainstaluj `naps2-7.4.3-win-x86.msi` |
| Skanowanie wisi na „Starting scan" | sterownik wymaga pulpitu (dotyczy pracy zdalnej/usług) | uruchom NAPS2 normalnie z pulpitu, nie jako usługa |
| Skan jest czarny/pusty | dokument na szybie złą stroną | kartka stroną zadrukowaną **do dołu** |

---

## 7. Wersja skrócona (dla kogoś, kto wie, co robi)

1. `ping 192.168.50.17` (i dla pewności `Test-NetConnection 192.168.50.17 -Port 9100`) —
   potwierdź, że drukarka odpowiada.
2. Pobierz ZIP sterownika (22 049 300 B, MD5 `D36F00E804CD10F30B5D7412BD117025`) i rozpakuj.
3. Uruchom `Setup.exe` jako administrator → **Sieć** → wybierz `Samsung M267x 287x Series`.
   Gdyby nie znalazł: Ustawienia → Drukarki → **Dodaj ręcznie → TCP/IP** → `192.168.50.17`
   → **Dysk...** → `SPL_PCL\ssa6m.inf`.
4. Drukuj stronę testową, sprawdź, że licznik stron urządzenia wzrósł o 1.
5. Skaner: zainstaluj pakiet Scanner (27 492 824 B), potem **NAPS2 7.4.3 x86** — 64-bitowe
   programy nie widzą tego sterownika TWAIN.

---

## 8. Linux (skrót, nie testowane na tej konkretnej maszynie)

Jeśli komputer docelowy ma Linuksa, nie instaluj sterowników Samsunga — użyj sterownika wbudowanego:

```
sudo apt install cups printer-driver-splix
# potem w przeglądarce: http://localhost:631 → Administration → Add Printer
#   Device: AppSocket/HP JetDirect → socket://192.168.50.17:9100
#   Make: Samsung → Model: Samsung M267x 287x (lub najbliższy SPL)
lp -d <nazwa_kolejki> /usr/share/cups/data/testprint
```

Skanowanie po sieci na Linuksie z tym modelem wymaga sterownika SANE dla Samsunga i nie było
weryfikowane w tej konfiguracji — najprościej sprawdzić `scanimage -L` po instalacji `sane-utils`
i pakietu backendu Samsung, jeśli jest dostępny w dystrybucji.

---

*Dokument opracowany na podstawie rzeczywistej konfiguracji: wydruk potwierdzony licznikiem sprzętowym
3987 → 3988, skan potwierdzony obrazem strony w NAPS2. Dane urządzenia w tym dokumencie są
**rzeczywiste**, nie przykładowe: IP `192.168.50.17`, nazwa `SEC001599DD4762`,
MAC `00-15-99-DD-47-62`, S/N `ZD0NBJDD30066P`, brama `192.168.50.1`.
Sprawdzone ponownie 2026-11-09: ping OK, porty 80/515/631/9100 otwarte, 443 zamknięty,
SNMP potwierdza model i numer seryjny. Jeśli Twój egzemplarz ma inny adres, ustal go krokiem 1.*
