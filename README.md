# Rejestr uszkodzeń wózków

Aplikacja do rejestrowania uszkodzeń wózków widłowych na magazynie: zgłoszenia (wózek, data, pracownik, rodzaj uderzenia, opis), statusy napraw, stan floty na żywo i mail do serwisu przez Outlooka.

- `docs/` — wersja na GitHub Pages (baza danych i logowanie: Firebase).
- `app/` — pierwsza wersja jako artefakt claude.ai.

## Uruchomienie (jednorazowo, ok. 15 minut)

### 1. Firebase (baza danych + logowanie, darmowy plan Spark)
1. Wejdź na https://console.firebase.google.com → **Dodaj projekt** (np. `rejestr-wozkow`), Google Analytics można wyłączyć.
2. **Build → Firestore Database → Utwórz bazę danych** → lokalizacja `eur3 (europe-west)` → tryb produkcyjny.
3. Zakładka **Reguły** → wklej zawartość pliku `firestore.rules` z tego repozytorium → **Opublikuj**.
4. **Build → Authentication → Rozpocznij → Email/Password** → włącz (bez „Email link”).
5. **Authentication → Settings → User actions** → odznacz **Enable create (sign-up)**, żeby nikt obcy nie założył sobie konta.
6. **Authentication → Users → Add user** → dodaj e-mail i hasło dla siebie i każdego specjalisty.
7. **Authentication → Settings → Authorized domains** → dodaj `czozen-source.github.io`.
8. **Ustawienia projektu (zębatka) → Twoje aplikacje → ikona `</>`** → zarejestruj aplikację webową → skopiuj obiekt `firebaseConfig` i wpisz jego wartości do `docs/firebase-config.js`.

### 1a. Moderatorzy (kto może zmieniać listy pracowników, wózków, rodzajów uderzeń i ustawienia maila)
1. **Firestore Database → Dane → Rozpocznij kolekcję** → ID kolekcji: `config`.
2. ID dokumentu: `roles`, pole `moderators`, typ **array**, elementy typu string: adresy e-mail moderatorów (małymi literami, tak jak konto w Authentication).
3. Zakładka **Reguły** → wklej aktualną zawartość `firestore.rules` → **Opublikuj**.

Specjaliści (pozostali zalogowani) dodają zgłoszenia i zmieniają statusy, ale nie widzą zakładki Ustawienia, nie mogą zmieniać list ani usuwać zgłoszeń. Pracownika w formularzu wybierają przez wyszukiwarkę (min. 2 litery) — pełna lista nie jest wyświetlana.

### 2. GitHub Pages
1. Repozytorium → **Settings → Pages**.
2. **Source: Deploy from a branch**, branch z plikami (`main` po scaleniu albo `claude/happy-planck-oank1g`), folder **`/docs`** → **Save**.
3. Po 1–2 minutach aplikacja będzie pod adresem `https://czozen-source.github.io/ch0sen/`.

GitHub Pages dla repozytorium prywatnego wymaga płatnego planu GitHub (Pro/Team). Na darmowym planie repozytorium musi być publiczne — w kodzie nie ma danych pracowników ani zgłoszeń (są tylko w Firebase, za logowaniem).

### 3. Pierwsze uruchomienie aplikacji
Zaloguj się i w **Ustawieniach**:
- **Powiadomienia e-mail** — adres grupy serwisowej (i opcjonalnie DW).
- **Import z Excela** — wklej kolumny „Imię i nazwisko / Dział” (Pracownicy) oraz „Nr wózka / Typ” (Wózki).
- **Rodzaje uderzeń** — kliknij „Dodaj domyślne rodzaje uderzeń” albo dodaj własne.

## Rozpiska wózków
Zakładka **Rozpiska** odwzorowuje arkusz „Rozpiska i kontrola wózków”: na każdy dzień 3 zmiany obok siebie (godziny i specjalista zmiany), a w nich stanowiska — Rozładunek zwrotów, Sprawdzanie zwrotów, Opakowania, Zadania dodatkowe, Biuro — z kolumnami Imię i nazwisko / Wózek / Uwagi. Uzupełnia każdy zalogowany (nazwisko wyszukuje się po 2 literach); aplikacja ostrzega, gdy jeden wózek ma dwie osoby na tej samej zmianie albo gdy wózek jest wyłączony z pracy.
- **Drukuj rozpiskę** — A4 poziomo, trzy zmiany obok siebie, miejsce na podpisy.
- **Wczytaj rozpiski z pliku Excel** — karty nazwane datą (np. `15.10.2026`) z pliku w tym układzie.
- **Sprawdź wózek** — kto jeździł danym wózkiem w zakresie dat, na jakiej zmianie i stanowisku, oraz usterki z tego dnia i dnia następnego.
- Zmiany (nazwy, godziny) i stanowiska (nazwy, liczba wierszy) ustawia moderator w Ustawieniach.
- Zgłoszenie uszkodzenia ma pola z rejestru: zmiana, co uszkodzone, kiedy powstało (zgłoszone przez sprawcę / zastane przy przejęciu); operatora podpowiada rozpiska.

## Mail do serwisu
Po zapisaniu zgłoszenia i po każdej zmianie statusu pojawia się przycisk **„Wyślij mail do serwisu”**, który otwiera Outlooka (lub domyślny program pocztowy) z wypełnionym adresem, tematem i treścią. Opcja „od razu otwórz Outlooka” w ustawieniach otwiera maila automatycznie po zapisaniu zgłoszenia. Mail wysyła osoba zgłaszająca ze swojej skrzynki.

## Raport zmiany (`docs/raport.html`)
**Wersja testowa: rejestr wózków jest dostępny tylko dla konta czozen@gmail.com; raport zmiany – dla wszystkich zalogowanych** (pozostałe konta po zalogowaniu w rejestrze trafiają od razu do raportu zmiany). Warunek w `docs/index.html`, stała `OWNER` w `docs/raport.html` (usuwanie raportów) i `isOwner()` w `firestore.rules`. Po zmianie `firestore.rules` wklej je w konsoli Firebase → Firestore → Reguły → Opublikuj.

Formularz raportu po zmianie dla specjalistów: `https://czozen-source.github.io/ch0sen/raport.html` (link „Raport zmiany →” jest też w nagłówku rejestru). Logowanie tym samym kontem co rejestr.

- Nie da się wysłać raportu z pustym polem; „ok”, „brak” itp. nie przechodzą jako odpowiedź.
- Bilans naczep i butelek liczy się sam; gdy się nie zgadza, trzeba podać powód.
- Sekcja 0 pokazuje, co zostawił poprzednik. Gdy stan na start różni się od jego stanu na koniec, specjalista musi opisać różnicę.
- Nadawca (adres e-mail konta) jest u góry raportu. Po zapisaniu formularz się czyści.
- Redlink: tygodniowy licznik palet przerzutów (pon–nd), każdy dopisuje swoje palety, limit 15 palet na dzień.
- Mail jest kolorowy: zielone = OK, czerwone = do uwagi. Temat ma [CZERWONY]/[ŻÓŁTY] przy takim statusie zmiany.
- Raporty: kolekcja `shift_reports`, plan Redlink: kolekcja `redlink`. Historia: wszystkie raporty i podsumowanie 7 dni na specjalistę.

### Jednorazowo
1. Firebase → Firestore Database → **Reguły** → wklej aktualny `firestore.rules` (reguły `shift_reports` i `redlink`) → **Opublikuj**. Bez tego raporty się nie zapisują.
2. Zaloguj się na `raport.html` jako moderator → **Ustawienia** → wpisz Do i DW.

### Automatyczna wysyłka kolorowego maila (EmailJS, darmowe do 200 maili/mies.)
Bez tego po kliknięciu „Wyślij raport” jest przycisk „Kopiuj raport z kolorami” – wklejasz go do nowego maila w Outlooku.
1. Załóż konto na https://www.emailjs.com.
2. **Email Services → Add New Service** → podłącz skrzynkę, z której mają wychodzić raporty → skopiuj **Service ID**.
3. **Email Templates → Create New Template**:
   - To Email: `{{to_email}}`, Cc: `{{cc_email}}`, Reply To: `{{reply_to}}`, From Name: `{{from_name}}`
   - Subject: `{{subject}}`
   - Content: przełącz edytor na **Code / HTML** i wpisz dokładnie `{{{html}}}` (trzy klamry – wtedy idą kolory)
   - zapisz i skopiuj **Template ID**.
4. **Account → General** → skopiuj **Public Key**. W **Account → Security** dodaj domenę `czozen-source.github.io` do dozwolonych.
5. Aplikacja → **Ustawienia** raportu → wpisz Service ID, Template ID i Public Key → Zapisz.

## Budżet Kokosza (`docs/budzet.html`)
Osobisty budżet: `https://czozen-source.github.io/ch0sen/budzet.html` (widżet: `…/budzet.html#widget`). Logowanie e-mailem i hasłem przez ten sam projekt Firebase co rejestr. **Dostęp mają tylko czozen@gmail.com i annakurnik90@gmail.com**, każde ze swoimi osobnymi danymi (`budgets/{uid}`). Lista jest w trzech miejscach: `BUDGET_USERS` w `budzet/kokosza.html` (i `docs/budzet.html`), `isBudgetUser()` w `firestore.rules`, `BUDGET_ONLY` w `docs/raport.html`. Każde inne konto dostaje „To konto nie ma dostępu do tej strony” i jest wylogowane, a baza odmawia mu odczytu. Konto annakurnik90@gmail.com nie widzi raportu zmiany, Redlinka ani rejestru wózków.
- Konto dla Anny: Authentication → Users → **Add user** (annakurnik90@gmail.com + hasło tymczasowe); hasło zmieni sama przez „Nie pamiętam hasła”.
- Jednorazowo: wklej aktualne `firestore.rules` w konsoli Firebase → Firestore → Reguły → **Opublikuj** (dochodzi blok `budgets`).
- Źródło jest w `budzet/kokosza.html`; `docs/budzet.html` to ten sam kod z nagłówkiem strony i skryptami Firebase.

## Awizacja opakowań (`docs/opakowania.html`)
Codzienna obsługa wydań palet i opakowań: `https://czozen-source.github.io/ch0sen/opakowania.html`. **Dostęp tylko dla konta czozen@gmail.com** (stała `OWNER` w `docs/opakowania.html`, `isOwner()` dla kolekcji `pk_*` w `firestore.rules`); inne konta są wylogowywane z komunikatem o braku dostępu, a link „Awizacja opakowań →” w raporcie zmiany widzi tylko właściciel.

- **Załadunki** — dwie sekcje: *Wydania EUR + pozostałe* (wózki, transportery, naprawa, sprzedaż) i *Wydania PUL* (CHEP, LPR, IPP, NDHP), każda z własnym podsumowaniem wydane/umówione. Lista awizacji dnia w kolejności załadunku (▲▼, „Numeruj wg okien”), statusy *Umówione → Przyjechał → Załadunek → Wydane* (oraz Anulowane / Nie przyjechał), wydanie z rzeczywistymi ilościami i nr dokumentu, mail z awizacją (kopiuje treść, a gdy w kontakcie jest e-mail, otwiera program pocztowy), druk planu dnia.
- **Harmonogram** — tydzień: okna × dni z obłożeniem (domyślnie 2 auta na 2-godzinne okno); klik w puste pole = nowa awizacja w tym oknie. Dłuższe okna (np. sobota 00:00-13:00) są w wierszu „inne / dłuższe”.
- **Salda** — codzienne wgranie pliku „Stan rozliczeń opakowań” (xlsx/xls/csv). Zmiana względem poprzedniego pliku, umówione odbiory, saldo po odbiorach, filtry i wyszukiwarka.
- **Stany** — poranny stan pustych opakowań na placu (te same kategorie co w arkuszu WYDANIA oraz Transportery: palety z transporterami Carlsberg, Kompania Piwowarska, Grupa Żywiec; plastik w stosach × 120/140). Zjazd ze sklepów = stan dziś − stan poprzedni + wydane w międzyczasie.
- **Co trzeba zrobić** (na górze Prognozy) — karty z działaniami, najpilniejsze pierwsze: ile aut umówić w danej kategorii i od kiedy przekroczony limit, brak dzisiejszego stanu placu, nieaktualny plik sald, dostawcy z saldem bez awizacji.
- W Stanach placu zmiana zapasu względem poprzedniego liczenia: **▲ wzrost na czerwono, ▼ spadek na zielono**.
- **Prognoza** — średni zjazd z 28 dni, prognoza stanu na 3/5/7/14 dni, dzień przekroczenia maks. stanu, liczba aut do zaplanowania, zjazd wg dnia tygodnia.
- Kolory grup opakowań wszędzie (stany, tabele, harmonogram, ilości): EUR własne – sosna, CHEP – niebieski, LPR – czerwony, NDHP – zielony, IPP – brązowy, transportery – morski, wózki – szary, inne – fiolet. Grupę kategorii zmienia się w Ustawieniach.
- **Eksport do Excela** (przycisk „⤓ Excel”) na każdej zakładce: załadunki dnia, tydzień harmonogramu, salda (wszystkie przefiltrowane wiersze), stany i zjazdy, prognoza; w Ustawieniach eksport wszystkich awizacji i wydań z dowolnego okresu.
- Skróty: <kbd>N</kbd> nowa awizacja, ←/→ zmiana dnia, w formularzu <kbd>Enter</kbd> zapisuje. Podpowiedź firmy po kodzie SAP albo 2 literach nazwy (z listy DOSTAWCY i sald).

### Jednorazowo
1. Firebase → Firestore Database → **Reguły** → wklej aktualny `firestore.rules` (bloki `pk_*`) → **Opublikuj**.
2. Zaloguj się → **Ustawienia → Import z pliku „OPAKOWANIA AWIZACJE”** → wybierz plik `.xlsx`. Przenosi awizacje z arkusza WYDANIA (przeszłe jako wydane), poranne stany placu, listę DOSTAWCY i SALDA. Arkusz z loginami i hasłami portali **nie jest importowany**. Ponowny import tego samego pliku nadpisuje te same pozycje.
3. W **Ustawieniach** sprawdź przeliczniki „szt. na auto” i „maks. stan (auta)” dla kategorii — od nich zależy, ile aut prognoza każe planować.

Tryb próbny bez bazy (dane tylko w tej przeglądarce): `opakowania.html#lokalnie`.
Dane: `pk_dni/{RRRR-MM-DD}` (awizacje dnia + stan placu, jeden dokument na dzień), `pk_salda/{data}` (cały plik sald w jednym dokumencie), `pk_meta/salda`, `pk_meta/dostawcy`, `pk_config/main`.
