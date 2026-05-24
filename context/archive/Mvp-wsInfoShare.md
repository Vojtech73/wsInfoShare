## wsInfoShare - MVP

### Potrzeba biznesowa  
Aplikacja umożliwiająca tworzenie kart informacji przez Edytorów, które mogą być używane  i udostępniane (unikatowy URL dla karty informacji pre Odbiorca) przez Odbiorców. Jeden Odbiorcy maja dostęp do całej bazy publicznych Kart.
Edytor może być też Odbiorcą innych Edytorów. 
Obiorca, może dodawać Karty, ale tylko na własny użytek, czyli nie może ich udostępniać innym Odbiorcom. Żeby to robić, musi dostać rolę Edytora.
Edytor i Odbiorca to tenant - klient biznesowy identyfikowany po NIP lub innym oficjalnym ID.

### MVP
 - Panel admina.
 - Panel Edytora.
 - Panel Odbiorcy.
 - Zakładanie przez admina tenat'ów i nadawanie im ról - id po NIP.
 - Autentykacja w zewnętrznej usłudze, np. keycloak
 - Dodawanie użytkowników przez admina tenanta.
 - Logowanie przez użytkowników Edytora i Odbiorcy.
 - Założenie karty w panelu Edytora.
 - Karta informacji - możliwość dodawania atrybutów przez Edytorów z listy konfigurowanej przez admina.
 - Karty maja czas obowiązywania.
 - Status Karty - draft, published, outdated.
 - Nadawanie przez edytora zasięgu Karty Informacji: globalny, lokalny, prywatny.
 - Zasięg lokalny - przypisanie miast (+ zasięg, np 20km), w których karta jest dostępna.
 - Generator unikatowych URL dla Kart per user.
 - Publiczny widok udostępnionej przez odbiorcę karty.
 - Generator kodów QR z URL.
 - Wyszukiwarka Kart wg. różnych kryteriów.
 - Architektura pod automatyczne tłumaczenie treści Kart (AI) - statyczne po stronie edytora lub dynamiczne po wyświetleniu przez klienta (wybór języka).
 - Architektura pod multilanguage.
 
 ### Czego nie obejmuje MVP
  - Import treści karty z pliku.
  - Wdrożenie automatycznego tłumaczenia treści Kart.
  - Edytor WYSIWYG.
  - Raportowanie po stronie Edytora i Odbiorcy.
  - Wdrożenie wersji multilanguage.
  - RLS na bazie.
  - Uzupełnianie danych tenanta na podstawie NIP z baz, np. GUS (chyba, że będzie gotowa biblioteka prosta w implementacji).
  - Wersjonowanie kart.
  - Powiadomienia o nowych Kartach lub zakończeniu publikacji.
    