---
project: "wsInfoShare"
context_type: greenfield
created: 2026-05-24
updated: 2026-05-24
checkpoint:
  current_phase: 8
  phases_completed: [1, 2, 3, 4, 5, 6, 7]
  gray_areas_resolved:
    - topic: "zasięg kart w MVP"
      decision: "tylko globalny; lokalny i prywatny → v2"
    - topic: "atrybuty kart"
      decision: "stały zestaw w MVP; konfigurowalne przez admina → v2"
    - topic: "AI / multilanguage"
      decision: "pola tekstowe w v1; architektura AI/ML → v2"
    - topic: "rola przypisana do tenanta czy użytkownika"
      decision: "do tenanta; wszyscy użytkownicy tenanta dziedziczą rolę"
    - topic: "Admin — zakres widoczności"
      decision: "super-tenant; widzi wszystko"
    - topic: "mechanizm auth"
      decision: "otwarta kwestia — Keycloak wymieniony jako kandydat, decyzja w tech-stack"
    - topic: "szacunek tygodni MVP"
      decision: "otwarta kwestia — nie zdecydowano"
    - topic: "branża docelowa w MVP"
      decision: "platforma generyczna; brak zawężenia branżowego w MVP"
  frs_drafted: 16
  quality_check_status: warned
---

## Vision & Problem Statement

Firmy B2B (identyfikowane NIPem) mają dwa powiązane bóle:

**Edytor** (firma dostarczająca informacje): ręczna dystrybucja — informacje wysyłane przez e-mail, PDF, telefon. Zmiana informacji wymaga ponownego rozsyłania; wersje się gubią; brak kontroli nad tym kto widzi co i od kiedy do kiedy.

**Odbiorca** (firma konsumująca informacje): brak jednego miejsca z aktualnymi informacjami od swoich dostawców/partnerów; informacje dostępne tylko przez bezpośredni kontakt z Edytorem (telefon, e-mail), bez dostępu 24/7 do aktualnych danych.

Wgląd: nie istnieje rozwiązanie, które łączy model tenantowy B2B (weryfikacja NIP), zarządzanie zasięgiem kart informacji i mechanizm unikalnych URL/QR per Odbiorca. Świadoma abstrakcja branżowa na tym etapie.

## User & Persona

**Główna persona — Edytor (tenant)**: firma B2B zarejestrowana w Polsce, identyfikowana NIPem. Tworzy i publikuje Karty Informacji. Chce kontrolować, kto widzi daną informację i od kiedy do kiedy jest aktualna. Ból: ręczna dystrybucja przez e-mail/PDF/telefon.

**Persona drugorzędna — Odbiorca (tenant)**: firma B2B identyfikowana NIPem. Konsumuje i udostępnia Karty Informacji swoim klientom. Ból: brak jednego miejsca z aktualnymi danymi + brak 24/7 samoobsługowego dostępu do informacji dostawców.

**Relacja**: Edytor może być jednocześnie Odbiorcą kart innych Edytorów. Odbiorca może dodawać karty wyłącznie na własny użytek (bez możliwości udostępniania innym Odbiorcom). Żeby udostępniać karty, Odbiorca musi dostać rolę Edytora.

**Rola na poziomie tenanta**: wszyscy użytkownicy tenanta dziedziczą rolę swojego tenanta (Edytor lub Odbiorca).

**Admin (globalna rola platformy)**: super-tenant; widzi wszystko; tworzy tenantów, nadaje role, dodaje użytkowników.

## Access Control

Model wielotenantowy. Rola przypisana do **tenanta** (firmy), nie do indywidualnego użytkownika.

- **Admin** — rola globalna platformy. Tworzy tenantów (NIP), nadaje im role (Edytor/Odbiorca/obydwie), dodaje użytkowników do tenantów. Widzi treść kart wszystkich Edytorów. Konfiguruje listę dostępnych atrybutów kart (v2).
- **Edytor** — tenant z rolą Edytora. Jego użytkownicy: tworzą i zarządzają Kartami (draft/published/outdated), określają zasięg karty, widzą karty własne.
- **Odbiorca** — tenant z rolą Odbiorcy. Jego użytkownicy: przeglądają publiczne Karty, wyszukują Karty, generują unikalne URL per Karta, generują QR z URL, dodają Karty wyłącznie na własny użytek (prywatne, nieudostępnialne).

Mechanizm auth: **otwarta kwestia** (Keycloak/zewnętrzny IdP wymieniony jako kandydat; decyzja należy do etapu tech-stack). Wymaganie: uwierzytelnianie przez zewnętrzną usługę.

Publiczny widok karty (via unikalny URL/QR): dostępny bez logowania dla każdego posiadacza linku, ale tylko dla kart w statusie `published` i przed datą ważności.

## Success Criteria

### Primary
- Pełny flow E2E działa: Edytor publikuje Kartę → Odbiorca widzi ją w wyszukiwarce, generuje unikalny URL + QR → osoba skanująca QR widzi publiczny widok Karty bez logowania.
- Główny dowód wartości: **Karta może być udostępniona przez unikalny URL**.

### Secondary
- Wyszukiwarka Kart działa poprawnie wg. kryteriów dostępnych dla Odbiorcy.
- Karty mają widoczny status i datę ważności.

### Guardrails
- **Unikalność URL**: URL wygenerowany przez Odbiorcę A dla danej Karty musi być unikatowy i nieprzewidywalny; nie może być identyczny z URL Odbiorcy B dla tej samej Karty.
- **Prywatność**: Karty w statusie `draft` oraz karty dodane przez Odbiorcę na własny użytek nie są widoczne dla innych tenantów ani publicznie.
- **Ważność**: Karta po dacie ważności jest automatycznie niedostępna publicznie (stan `outdated`).

## Timeline acknowledgment

Szacunek tygodni: **otwarta kwestia** — nie zdecydowano podczas sesji kształtowania. Należy rozstrzygnąć przed startem implementacji.

Uproszczenia zaakceptowane w stosunku do pierwotnego zakresu z Mvp-wsInfoShare.md:
- Zasięg kart: tylko globalny (lokalny/prywatny → v2)
- Atrybuty kart: stały predefiniowany zestaw (konfigurowalne przez admina → v2)
- Architektura AI/multilanguage: notatka projektowa, nie implementowany feature (→ v2)

## Functional Requirements

### Administracja (platforma)

- FR-001: Admin platformy może utworzyć tenanta identyfikowanego ogólnym ID biznesowym (np. NIP lub inny oficjalny ID) i przypisać mu rolę (Edytor/Odbiorca/obydwie). Priorytet: must-have
  > Sokrates: Rozważono kontrargument "NIP ogranicza MVP do polskich firm". Rozwiązanie: zmieniono na ogólny ID biznesowy (NIP lub inny oficjalny identyfikator).

- FR-002: Admin platformy może dodać administratora tenanta. Priorytet: must-have

### Administracja (tenant)

- FR-003: Admin tenanta może dodawać i usuwać użytkowników w ramach swojego tenanta. Priorytet: must-have

### Uwierzytelnianie

- FR-004: Użytkownik może zalogować się przez zewnętrzną usługę uwierzytelniania (zewnętrzny IdP). Priorytet: must-have
  > Sokrates: Rozważono kontrargument "Keycloak/zewnętrzny IdP opóźni MVP". Rozwiązanie: zachowano — bezpieczeństwo i SSO to wymaganie od dnia 1 dla multi-tenant B2B.

### Karty Informacji — tworzenie i zarządzanie (Edytor)

- FR-005: Edytor może stworzyć Kartę Informacji z atrybutami: Nazwa, Opis, Data obowiązywania (od-do), Kategoria. Priorytet: must-have
- FR-006: Edytor może zmieniać status Karty ręcznie: draft → published → outdated. Priorytet: must-have
  > Sokrates: Rozważono kontrargument "karta published po dacie ważności jest myląca". Rozwiązanie: status tylko manualny — Edytor odpowiada za aktualność; publiczny widok wyświetla datę ważności czytelnie dla odbiorcy końcowego.
- FR-007: Edytor może określić zasięg Karty jako globalny. Priorytet: must-have
- FR-008: Edytor może przeglądać listę swoich Kart. Priorytet: must-have
- FR-009: Edytor może edytować Kartę (atrybuty, status, zasięg). Priorytet: must-have
- FR-010: Edytor może archiwizować Kartę (zamiast trwałego usunięcia). Po archiwizacji URL nadal działa i wyświetla komunikat "karta niedostępna". Priorytet: must-have
  > Sokrates: Rozważono kontrargument "wydrukowane QR kody stają się bezużyteczne po usunięciu karty". Rozwiązanie: archiwizacja zamiast hard delete — URL pozostaje aktywny z komunikatem.

### Wyszukiwanie i przeglądanie (Odbiorca)

- FR-011: Odbiorca może wyszukiwać publiczne Karty według kryteriów: nazwa, opis, kategoria, data ważności. Priorytet: must-have

### URL i QR (Odbiorca)

- FR-012: Odbiorca może wygenerować unikalny URL dla wybranej Karty (jeden URL per Odbiorca per Karta; regeneracja nadpisuje poprzedni URL). Priorytet: must-have
  > Sokrates: Rozważono kontrargument "nieograniczone URLe na tę samą kartę = bałagan śledzenia". Rozwiązanie: jeden URL per Odbiorca per Karta; regeneracja nadpisuje stary link.
- FR-013: Odbiorca może wygenerować kod QR z unikalnym URL. Priorytet: must-have
- FR-014: Odbiorca widzi listę Kart, dla których wygenerował URL (historia linków). Priorytet: must-have

### Publiczny widok

- FR-015: Każda osoba posiadająca unikalny URL może wyświetlić publiczny widok Karty bez logowania. Widok wyświetla wyraźnie datę ważności karty. Priorytet: must-have
  > Sokrates: Rozważono kontrargument "karta published po dacie ważności = złudna gwarancja aktualności". Rozwiązanie: publiczny widok wyświetla "ważna do: [data]" — klient końcowy ocenia aktualność sam.
- FR-016: URL prowadzący do zarchiwizowanej Karty wyświetla komunikat "karta niedostępna". Priorytet: must-have

### Nice-to-have (poza MVP)

- FR-017: Odbiorca może dodać Kartę na własny użytek (prywatna, nieudostępnialna). Priorytet: nice-to-have

## User Stories

### US-01: Edytor publikuje Kartę i Odbiorca ją udostępnia przez QR

- **Given** zalogowany użytkownik tenanta Edytora z aktywną kartą w statusie `published`
- **When** Odbiorca wyszukuje i znajduje tę Kartę, a następnie generuje unikalny URL i QR
- **Then** osoba skanująca QR widzi publiczny widok Karty bez logowania, wraz z datą ważności

#### Acceptance Criteria
- Kartę w statusie `draft` lub `outdated` Odbiorca nie widzi w wyszukiwarce (widoczne tylko `published`)
- Każdy URL wygenerowany przez Odbiorcę A dla Karty X jest nieprzewidywalny i różny od URL Odbiorcy B dla tej samej Karty X
- Publiczny widok wyświetla co najmniej: Nazwę, Opis, Kategorię, "Ważna do: [data]"

## Business Logic

Aplikacja kontroluje dwie powiązane reguły domenowe:

1. **Reguła widoczności**: Karta jest dostępna dla Odbiorcy wtedy i tylko wtedy, gdy jej status to `published`, jej zasięg obejmuje kontekst Odbiorcy (w MVP: zasięg globalny oznacza dostępność dla wszystkich Odbiorców), oraz nie jest zarchiwizowana.

2. **Reguła identyfikacji**: Platforma generuje unikalny, kryptograficznie nieprzewidywalny identyfikator (URL i QR) per Odbiorca per Karta — każdy Odbiorca dostaje swój własny link do tej samej Karty, który nie może być odgadnięty przez iterację ani przez znalezienie URL innego Odbiorcy.

Dane wejściowe reguły widoczności: status Karty (ustawiany manualnie przez Edytora), zasięg Karty (globalny/lokalny/prywatny), status archiwizacji. Wynik: Karta widoczna lub niewidoczna w wyszukiwarce Odbiorcy.

Dane wejściowe reguły identyfikacji: tożsamość Odbiorcy + identyfikator Karty. Wynik: unikalny token URL, który nie powtarza się dla żadnej innej pary Odbiorca-Karta.

Użytkownik napotyka obie reguły w przepływie: Edytor publikuje kartę → reguła widoczności decyduje czy Odbiorca ją zobaczy; Odbiorca generuje URL → reguła identyfikacji decyduje jaki token dostanie.

## Non-Functional Requirements

- Unikalny URL karty jest kryptograficznie nieprzewidywalny: żadne dwa URLe per para (Odbiorca, Karta) nie są identyczne; tokena nie można odgadnąć przez iterację ani enumerację.
- Publiczny widok Karty ładuje się w czasie postrzeganym przez użytkownika poniżej 2 sekund w standardowych warunkach sieci mobilnej.
- Platforma działa poprawnie na dwóch ostatnich wersjach głównych przeglądarek: Chrome, Firefox, Safari, Edge (desktop i mobile).
- Dane jednego tenanta są niewidoczne dla innych tenantów: Odbiorca A nie może zobaczyć kart w statusie `draft`, kart zarchiwizowanych ani danych wewnętrznych tenanta B.
- Platforma jest zaprojektowana pod skalowanie: architektura powinna umożliwiać wzrost liczby tenantów i użytkowników bez przepisywania kluczowych komponentów.

## Non-Goals

- **Brak zasięgu lokalnego (miasto + promień km) i prywatnego** — zasięg lokalny i prywatny to v2+; MVP obsługuje wyłącznie zasięg globalny.
- **Brak raportowania** — brak statystyk wyświetleń, kliknięć ani skanów QR per Edytor i Odbiorca w MVP; analytics to v2.
- **Brak wersjonowania Kart** — edycja Karty nadpisuje wersję; historia zmian nie jest śledzona w MVP.
- **Brak powiadomień** — brak e-maili, push ani alertów o nowych Kartach, zmianach statusów ani wygaśnięciu w MVP.
- **Brak importu treści z pliku** — Karty tworzone wyłącznie ręcznie w panelu Edytora; brak importu CSV/Excel.
- **Brak edytora WYSIWYG** — pola tekstowe (plain text lub podstawowe formatowanie); brak edytora wizualnego.
- **Brak implementacji tłumaczeń AI i multilanguage** — pola tekstowe w v1; architektura AI/ML i i18n to v2.
- **Brak uzupełniania danych tenanta z rejestrów (GUS/NIP)** — Admin wpisuje dane tenanta ręcznie.
- **Brak RLS na bazie danych** — izolacja tenantów realizowana na poziomie aplikacji w MVP; row-level security to v2.

## Open Questions

1. **Typ produktu** — web-app vs. web-app + headless API — nie zdecydowano podczas sesji kształtowania. Decyzja: etap wyboru stosu technologicznego.
2. **Szacunek tygodni MVP** — nie zdecydowano; projektant szacuje po godzinach, bez twardego deadline'u. Należy rozstrzygnąć przed startem implementacji.
3. **Mechanizm auth** — zewnętrzny IdP (Keycloak lub inny) jest wymaganiem, ale konkretny produkt nie wybrany. Decyzja: etap wyboru stosu.
4. **Zarządzanie wygasłymi kartami** — status jest manualny (FR-006), ale karta może być `published` po dacie ważności. Pytanie: czy publiczny widok po prostu wyświetla "ważna do: [data]" (zdecydowano: tak), czy system powinien automatycznie zmieniać status na `outdated` gdy data minie? Otwarte: czy potrzebny jest mechanizm alertów dla Edytora o wygasających kartach?
5. **Skala docelowa** — użytkownik wskazał na "ma być skalowalne" bez konkretnych liczb; architektura powinna wspierać wzrost. Konkretne cele wydajnościowe (QPS, liczba tenantów) — do ustalenia.

## Quality cross-check

Kontrola jakości przeprowadzona 2026-05-24. Status: **warned**.

- ✅ Kontrola dostępu — obecna (model wielotenantowy, 3 poziomy ról)
- ✅ Logika biznesowa — obecna (reguła widoczności + reguła identyfikacji)
- ✅ Artefakty projektu — shape-notes.md z poprawnym checkpoint
- ⚠️ Potwierdzenie kosztów czasowych — `mvp_weeks` nieznane; brak twardego deadline; użytkownik świadom pracy po godzinach. Konsekwencja: `timeline_budget.mvp_weeks: null` w PRD; punkt 2 w Open Questions.
- ✅ Non-Goals — 9 wpisów
- N/A Zachowane zachowanie — projekt greenfield

## Forward: tech-stack

- Wymaganie: zewnętrzny IdP dla uwierzytelniania (Keycloak wymieniony jako kandydat)
- Wymaganie: skalowalna architektura (brak monolitu trudnego do skalowania)
- Publiczny widok Karty — kluczowa ścieżka dla wydajności (QR scan → widok < 2s)
- Generowanie unikalnych tokenów URL — kryptograficznie bezpieczne (UUID v4 lub token o entropii ≥ 128 bitów)
