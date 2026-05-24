---
project: "wsInfoShare"
version: 1
status: draft
created: 2026-05-24
context_type: greenfield
product_type: "# TODO: product_type — see Open Questions"
target_scale:
  users: large
  qps: "# TODO: target_scale.qps — see Open Questions"
  data_volume: "# TODO: target_scale.data_volume — see Open Questions"
timeline_budget:
  mvp_weeks: null
  hard_deadline: null
  after_hours_only: true
---

## Vision & Problem Statement

Firmy B2B działające jako dostawcy informacji (Edytorzy) tracą czas na ręczną dystrybucję informacji — przez e-mail, PDF i telefon. Każda zmiana wymaga ponownego rozsyłania; starsze wersje krążą bez kontroli; Edytor nie może precyzyjnie zarządzać tym, kto widzi jakie informacje i w jakim oknie czasowym. Firmy odbierające te informacje (Odbiorcy) nie mają samoobsługowego dostępu 24/7 do aktualnych danych od swoich dostawców — każde zapytanie wymaga bezpośredniego kontaktu.

Luka rynkowa polega na braku platformy, która łączy trzy elementy jednocześnie: model tenantowy B2B z identyfikacją firm po oficjalnym numerze rejestracyjnym (zapewniający wiarygodność tożsamości), zarządzanie zasięgiem i cyklem życia kart informacji (Edytor kontroluje co, dla kogo i od kiedy do kiedy) oraz mechanizm unikalnego URL per Odbiorca per Karta umożliwiający bezpieczne udostępnianie informacji klientom końcowym przez QR.

## User & Persona

**Główna persona — Edytor (tenant)**: firma B2B zarejestrowana i identyfikowana oficjalnym numerem rejestracyjnym (np. NIP lub inny oficjalny identyfikator). Tworzy i publikuje Karty Informacji dla zewnętrznych odbiorców. Dotknięty bólem: każda zmiana informacji wymaga ręcznego rozsyłania; brak kontroli nad tym, kto widzi aktualną wersję i do kiedy jest ona ważna.

**Persona drugorzędna — Odbiorca (tenant)**: firma B2B identyfikowana oficjalnym numerem rejestracyjnym. Konsumuje Karty Informacji od Edytorów i udostępnia je swoim klientom końcowym. Dotknięty bólem: brak jednego miejsca z aktualnymi informacjami od dostawców; konieczność bezpośredniego kontaktu zamiast samoobsługi 24/7.

**Relacja między personami**: Edytor może jednocześnie być Odbiorcą kart innych Edytorów. Odbiorca może tworzyć Karty wyłącznie na własny użytek wewnętrzny (nieudostępnialne innym Odbiorcom); aby udostępniać karty publicznie, Odbiorca musi otrzymać rolę Edytora.

**Model tenantowy**: rola (Edytor/Odbiorca) przypisana jest do tenanta (firmy), nie do indywidualnego użytkownika — wszyscy użytkownicy tenanta dziedziczą rolę swojej firmy.

**Admin (rola globalna platformy)**: zarządza tenantami, rolami i administratorami tenantów; widzi treść wszystkich Kart wszystkich Edytorów.

## Success Criteria

### Primary
- Pełny flow end-to-end działa: Admin zakłada tenantów Edytora i Odbiorcy → Edytor tworzy i publikuje Kartę Informacji → Odbiorca wyszukuje Kartę, generuje unikalny URL i QR → osoba skanująca QR widzi publiczny widok Karty bez logowania.
- Główny dowód wartości: Karta Informacji może być udostępniona przez unikalny URL wygenerowany przez Odbiorcę.

### Secondary
- Wyszukiwarka Kart zwraca poprawne wyniki według dostępnych kryteriów (nazwa, opis, kategoria, data ważności).
- Odbiorca może powrócić do historii wygenerowanych linków i ponownie pobrać QR dla dowolnej Karty.

### Guardrails
- Każdy URL wygenerowany dla Karty przez Odbiorcę A jest nieprzewidywalny i różny od URL wygenerowanego przez Odbiorcę B dla tej samej Karty.
- Karty w statusie `draft` oraz Karty dodane przez Odbiorcę na własny użytek nie są widoczne dla innych tenantów ani dostępne publicznie przez URL.
- Publiczny widok Karty wyświetla datę ważności (może być bezterminowa), umożliwiając klientowi końcowemu samodzielną ocenę aktualności informacji.

## User Stories

### US-01: Edytor publikuje Kartę i Odbiorca ją udostępnia przez QR

- **Given** zalogowany użytkownik tenanta Edytora z Kartą Informacji w statusie `published`
- **When** Odbiorca wyszukuje i znajduje tę Kartę, a następnie generuje unikalny URL i QR
- **Then** osoba skanująca QR widzi publiczny widok Karty bez logowania, wraz z datą ważności

#### Acceptance Criteria
- Karta w statusie `draft` lub `outdated` nie jest widoczna w wyszukiwarce Odbiorcy
- Każdy URL wygenerowany przez Odbiorcę A dla Karty X jest nieprzewidywalny i różny od URL Odbiorcy B dla tej samej Karty X
- Publiczny widok wyświetla co najmniej: Nazwę, Opis, Kategorię, "Ważna do: [data]"

## Functional Requirements

### Administracja (platforma)

- FR-001: Admin platformy może utworzyć tenanta identyfikowanego oficjalnym ID biznesowym (np. NIP lub inny oficjalny identyfikator rejestracyjny) i przypisać mu rolę (Edytor/Odbiorca/obydwie). Priorytet: must-have
  > Sokrates: Rozważono kontrargument "NIP ogranicza MVP do polskich firm". Rozwiązanie: zmieniono na ogólny ID biznesowy (NIP lub inny oficjalny identyfikator).

- FR-002: Admin platformy może dodać administratora tenanta. Priorytet: must-have

### Administracja (tenant)

- FR-003: Admin tenanta może dodawać i usuwać użytkowników w ramach swojego tenanta. Priorytet: must-have

### Uwierzytelnianie

- FR-004: Użytkownik może zalogować się przez zewnętrzną usługę uwierzytelniania. Priorytet: must-have
  > Sokrates: Rozważono kontrargument "zewnętrzna usługa auth opóźni MVP". Rozwiązanie: zachowano — centralne zarządzanie tożsamością to wymaganie od dnia 1 dla wielotenantowego B2B.

### Karty Informacji — tworzenie i zarządzanie (Edytor)

- FR-005: Edytor może stworzyć Kartę Informacji z atrybutami: Nazwa, Opis, Data obowiązywania (od-do), Kategoria. Priorytet: must-have
- FR-006: Edytor może zmieniać status Karty ręcznie: draft → published → outdated. Priorytet: must-have
  > Sokrates: Rozważono kontrargument "karta published po dacie ważności jest myląca". Rozwiązanie: status tylko manualny — Edytor odpowiada za aktualność; publiczny widok wyświetla datę ważności czytelnie.
- FR-007: Edytor może określić zasięg Karty jako globalny. Priorytet: must-have
- FR-008: Edytor może przeglądać listę swoich Kart. Priorytet: must-have
- FR-009: Edytor może edytować Kartę (atrybuty, status, zasięg). Priorytet: must-have
- FR-010: Edytor może archiwizować Kartę. Po archiwizacji URL nadal działa i wyświetla komunikat "karta niedostępna". Priorytet: must-have
  > Sokrates: Rozważono kontrargument "wydrukowane QR kody stają się bezużyteczne po usunięciu karty". Rozwiązanie: archiwizacja zamiast trwałego usunięcia — URL pozostaje aktywny z komunikatem.

### Wyszukiwanie i przeglądanie (Odbiorca)

- FR-011: Odbiorca może wyszukiwać publiczne Karty według kryteriów: nazwa, opis, kategoria, data ważności. Priorytet: must-have

### URL i QR (Odbiorca)

- FR-012: Odbiorca może wygenerować unikalny URL dla wybranej Karty (jeden URL per Odbiorca per Karta; regeneracja nadpisuje poprzedni URL). Priorytet: must-have
  > Sokrates: Rozważono kontrargument "nieograniczone URLe na tę samą kartę = bałagan śledzenia". Rozwiązanie: jeden URL per Odbiorca per Karta; regeneracja nadpisuje.
- FR-013: Odbiorca może wygenerować kod QR z unikalnym URL. Priorytet: must-have
- FR-014: Odbiorca widzi listę Kart, dla których wygenerował URL (historia linków). Priorytet: must-have

### Publiczny widok

- FR-015: Każda osoba posiadająca unikalny URL może wyświetlić publiczny widok Karty bez logowania. Widok wyświetla wyraźnie datę ważności Karty. Priorytet: must-have
  > Sokrates: Rozważono kontrargument "karta published po dacie ważności = złudna gwarancja aktualności". Rozwiązanie: publiczny widok wyświetla "ważna do: [data]" — klient końcowy ocenia aktualność samodzielnie.
- FR-016: URL prowadzący do zarchiwizowanej Karty wyświetla komunikat "karta niedostępna". Priorytet: must-have

### Nice-to-have (poza MVP)

- FR-017: Odbiorca może dodać Kartę na własny użytek (prywatna, nieudostępnialna). Priorytet: nice-to-have

## Non-Functional Requirements

- Każdy URL Karty wygenerowany per Odbiorca per Karta jest niepowtarzalny i niemożliwy do odgadnięcia przez iterację ani przez obserwację URL innego Odbiorcy dla tej samej Karty.
- Publiczny widok Karty (otwierany przez skan QR) ładuje się w czasie postrzeganym przez użytkownika poniżej 2 sekund w standardowych warunkach sieci mobilnej.
- Platforma działa poprawnie na dwóch ostatnich wersjach głównych przeglądarek: Chrome, Firefox, Safari, Edge (desktop i mobile).
- Dane jednego tenanta są niewidoczne dla użytkowników innych tenantów: Karty w statusie `draft`, Karty zarchiwizowane oraz dane wewnętrzne tenanta nie są dostępne dla innych tenantów.
- Architektura platformy pozwala na wzrost liczby tenantów i użytkowników bez konieczności przepisywania kluczowych komponentów (zaprojektowana pod skalowanie).

## Business Logic

Karta Informacji jest dostępna dla Odbiorcy wyłącznie wtedy, gdy Edytor jawnie ją opublikował i nie zarchiwizował, a każde udostępnienie generuje dla Odbiorcy własny, unikalny identyfikator URL nieprzewidywalny przez osobę trzecią.

**Reguła widoczności**: Karta jest widoczna dla Odbiorcy w wyszukiwarce i dostępna przez unikalny URL wtedy i tylko wtedy, gdy spełnione są jednocześnie trzy warunki: (1) status Karty to `published` — ustawiony ręcznie przez Edytora, (2) zasięg Karty obejmuje kontekst Odbiorcy (w MVP: zasięg globalny obejmuje wszystkich Odbiorców), (3) Karta nie jest zarchiwizowana. Edytor samodzielnie zarządza wszystkimi stanami; system nie zmienia statusu automatycznie.

**Reguła identyfikacji**: Dla każdej pary (Odbiorca, Karta) platforma generuje dokładnie jeden unikalny identyfikator URL. Identyfikator ten jest charakterystyczny wyłącznie dla tej pary — żaden inny Odbiorca dla tej samej Karty nie otrzyma tego samego identyfikatora, a żadna osoba trzecia nie może go odgadnąć przez wyliczenie. Regeneracja URL nadpisuje poprzedni identyfikator dla tej pary.

Użytkownik napotyka obie reguły w przepływie produktu: Edytor zmienia status na `published` → reguła widoczności sprawia, że Karta pojawia się w wyszukiwarce Odbiorcy; Odbiorca generuje URL → reguła identyfikacji zapewnia, że link jest unikalny dla tej pary Odbiorca-Karta i może być bezpiecznie udostępniony klientowi końcowemu przez QR.

## Access Control

Model wielotenantowy. Rola przypisana do tenanta (firmy), nie do indywidualnego użytkownika — wszyscy użytkownicy danego tenanta dziedziczą rolę swojej firmy.

**Admin platformy** (rola globalna): tworzy tenantów (z oficjalnym ID biznesowym), przypisuje im role (Edytor/Odbiorca/obydwie), dodaje administratorów tenantów; ma dostęp do treści Kart wszystkich Edytorów.

**Admin tenanta** (rola w ramach tenanta): zarządza użytkownikami swojego tenanta (dodaje/usuwa); korzysta z tego samego panelu co rola tenanta z dodatkową sekcją zarządzania użytkownikami.

**Edytor** (tenant z rolą Edytora): użytkownicy tworzą, edytują, zarządzają statusem i archiwizują Karty Informacji należące do ich tenanta; widzą wyłącznie własne Karty w panelu Edytora.

**Odbiorca** (tenant z rolą Odbiorcy): użytkownicy wyszukują i przeglądają publiczne Karty wszystkich Edytorów; generują unikalne URL i QR dla wybranych Kart; widzą historię swoich wygenerowanych linków.

**Niezalogowany użytkownik (posiadacz URL)**: dostęp wyłącznie do publicznego widoku konkretnej Karty przez unikalny URL; brak dostępu do wyszukiwarki ani jakiegokolwiek innego widoku platformy.

Mechanizm uwierzytelniania: zewnętrzna usługa uwierzytelniania — konkretne rozwiązanie jest otwartą kwestią (patrz Open Questions nr 3).

## Non-Goals

- **Brak zasięgu lokalnego (miasto + promień km) i prywatnego** — MVP obsługuje wyłącznie zasięg globalny; zasięg lokalny i prywatny to v2+.
- **Brak raportowania** — brak statystyk wyświetleń, kliknięć ani skanowań QR per Edytor i Odbiorca; analytics to v2.
- **Brak wersjonowania Kart** — edycja Karty nadpisuje poprzednią treść; historia zmian nie jest śledzona w MVP.
- **Brak powiadomień** — brak e-maili ani alertów o nowych Kartach, zmianach statusów ani zbliżającym się wygaśnięciu w MVP.
- **Brak importu treści z pliku** — Karty tworzone wyłącznie ręcznie w panelu Edytora; brak importu masowego.
- **Brak edytora WYSIWYG** — pola tekstowe z podstawowym formatowaniem; brak edytora wizualnego w MVP.
- **Brak implementacji tłumaczeń i obsługi wielu języków** — pola tekstowe w MVP; mechanizm tłumaczenia i interfejs wielojęzyczny to v2.
- **Brak automatycznego uzupełniania danych tenanta z rejestrów publicznych** — Admin wpisuje dane tenanta ręcznie; integracja z bazami rejestracyjnymi to opcja v2.
- **Brak zabezpieczeń na poziomie wierszy bazy danych** — izolacja tenantów realizowana na poziomie aplikacji w MVP; zabezpieczenia na poziomie wierszy bazy danych to v2.

## Open Questions

1. **Typ produktu** (`product_type`) — web-app vs. web-app + headless API — nie zdecydowano podczas sesji kształtowania. Decyzja: etap wyboru stosu technologicznego. Blokuje: kształt architektury i dobór frameworków.

2. **Szacunek tygodni MVP** (`timeline_budget.mvp_weeks`) — nieznany; projekt realizowany po godzinach, bez twardego terminu. Należy określić przed startem implementacji.

3. **Mechanizm uwierzytelniania** — zewnętrzna usługa uwierzytelniania jest wymaganiem (FR-004), ale konkretne rozwiązanie nie zostało wybrane. Decyzja: etap wyboru stosu technologicznego.

4. **Zarządzanie wygasłymi Kartami** — ustalono, że publiczny widok wyświetla datę ważności i klient ocenia aktualność samodzielnie (FR-006, FR-015). Nierozstrzygnięte: czy system powinien alertować Edytora o Kartach, których data ważności zbliża się lub minęła? Jeśli tak — przez jaki kanał i z jakim wyprzedzeniem?

5. **Skala docelowa — QPS i wolumen danych** (`target_scale.qps`, `target_scale.data_volume`) — przyjęto wymaganie skalowania (NFR), ale konkretne cele liczbowe oraz docelowa liczba tenantów nie zostały określone. Do ustalenia przed wyborem stosu technologicznego.
