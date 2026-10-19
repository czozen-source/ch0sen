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
Zakładka **Rozpiska**: dzienny przydział pracowników do wózków na zmiany (nazwy zmian ustawia moderator w Ustawieniach). Każdy zalogowany układa rozpiskę w tabeli (nazwisko wyszukuje się po 2 literach), kopiuje ją z poprzedniego dnia albo wkleja z Excela (`nr wózka; nazwisko; zmiana`). Każdy może ją wydrukować. Sekcja „Kto korzystał z wózka” pokazuje i drukuje historię dla wózka i zakresu dat. Przy nowym zgłoszeniu i na liście zgłoszeń aplikacja podpowiada, kto wg rozpiski jeździł tym wózkiem w dniu zdarzenia.

## Mail do serwisu
Po zapisaniu zgłoszenia i po każdej zmianie statusu pojawia się przycisk **„Wyślij mail do serwisu”**, który otwiera Outlooka (lub domyślny program pocztowy) z wypełnionym adresem, tematem i treścią. Opcja „od razu otwórz Outlooka” w ustawieniach otwiera maila automatycznie po zapisaniu zgłoszenia. Mail wysyła osoba zgłaszająca ze swojej skrzynki.
