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
**Wersja testowa: cała aplikacja (rejestr wózków i raport zmiany) jest dostępna tylko dla konta czozen@gmail.com** – warunek w `docs/index.html`, stała `OWNER` w `docs/raport.html` oraz `signedIn()` i `isOwner()` w `firestore.rules`.

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
