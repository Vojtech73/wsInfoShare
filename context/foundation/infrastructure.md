---
project: ws-info-share
researched_at: 2026-06-05T00:00:00Z
recommended_platform: Hetzner VPS
runner_up: Railway
context_type: mvp
tech_stack:
  language: python
  framework: django
  runtime: python3.13
  package_manager: uv
---

## Rekomendacja

**Wdróż na Hetzner VPS (CX22) + Docker Compose + Caddy + Cloudflare proxy.**

Fly.io odrzucony po badaniach: brak free tier (deprecated 2024), minimalny koszt
~$20–25/miesiąc (VM + Managed Postgres). Railway (runner-up, 5/5 kryteriów) odrzucony:
$5/miesiąc to dolna granica + billing per-second CPU/RAM, realnie $10–15/miesiąc dla
działającego Django + Postgres. Użytkownik ma istniejącą infrastrukturę na Hetzner i
domenę testową na Cloudflare — nowy CX22 (~€4.50/miesiąc) z samodzielnie zarządzanym
PostgreSQL (Docker) i Caddy za Cloudflare proxy to najtańsza i najprostsza ścieżka
dla projektu eksperymentalnego prowadzonego "after hours".

## Porównanie platform

### Macierz punktacji

| Platforma | CLI-first | Managed | Agent docs | Stable API | MCP | Status Django |
|---|---|---|---|---|---|---|
| **Fly.io** | Zaliczone | Zaliczone | Częściowo | Zaliczone | Częściowo | GA (kontener) |
| **Railway** | Zaliczone | Zaliczone | Zaliczone | Zaliczone | **Zaliczone** (GA) | GA (Railpack) |
| **Render** | Częściowo | Zaliczone | Zaliczone | Częściowo | **Zaliczone** (GA) | GA (natywny) |
| **Vercel** | Zaliczone | Zaliczone | Zaliczone | Zaliczone | Częściowo (Beta) | Beta (serverless) |
| ~~Cloudflare~~ | — | — | — | — | — | **FAIL** (beta Python, brak transakcji) |
| ~~Netlify~~ | — | — | — | — | — | **FAIL** (brak persistent process) |

**Cloudflare Workers**: Python runtime open beta, Django obsługiwany wyłącznie przez
community adaptery (`django-cf`), brak transakcji na D1, Django Admin broken, `manage.py`
niedostępny przez CLI. Odrzucony na twardym filtrze Django-compatibility.

**Netlify**: Brak persistent WSGI process — architektonicznie niezgodny z Django.
Odrzucony na twardym filtrze.

**Vercel**: Python runtime w Beta, serverless model wymaga zewnętrznego PgBouncer (udokumentowany
connection exhaustion failure mode), brak persistent workers (Celery niemożliwy), `manage.py`
niedostępny bezpośrednio. Conditionally viable dla Django API-only, nieodpowiedni dla
full-featured multi-tenant B2B SaaS.

### Platformy na krótkiej liście

#### 1. Hetzner VPS + Docker Compose (Zalecana)

CX22 ~€4.50/miesiąc, pełna kontrola nad runtime, zero kosztów managed services. Docker
Compose + PostgreSQL na tym samym VM — wystarczające dla MVP przy niskim ruchu. Caddy jako
reverse proxy z Cloudflare Origin Certificate (Full strict mode). GitHub Actions CI/CD przez
SSH + GHCR. Użytkownik ma doświadczenie z Hetzner (istniejąca infrastruktura) — overhead
operacyjny jest znany i akceptowalny.

#### 2. Railway

Jedyna platforma z 5/5 kryteriów zaliczonych. GA MCP (oficjalny server, Claude integration
na `railway.com/agents/claude`), Postgres HA GA od marca 2026, CLI comprehensive
(`railway up`, `rollback`, `logs`, `run`). Railpack (nowy builder, GA marzec 2026)
auto-detects Django. Docs na GitHubie jako Markdown + raw .md URLs. Słabości: Python 3.13
w Railpack niezweryfikowany, EU West incidents (grudzień 2025), billing per-second może
zaskoczyć, MCP token z pełnym dostępem do konta.

#### 3. Render

Natywny Python 3.13 runtime GA, długotrwały WSGI/ASGI process, llms.txt + llms-full.txt.
GA MCP z 20+ narzędziami (sierpień 2025) i 3 oficjalnymi skillami dla Claude Code.
Słabości: rollback tylko przez API (brak `render rollback` w CLI), free tier spin-down po
15 min bezczynności (60s cold restart), free Postgres wygasa po 30 dniach, media files
znikają przy redeployu bez Persistent Disk (paid only).

## Weryfikacja krzyżowa anty-uprzedzeniowa: Hetzner VPS

*Uwaga: weryfikacja krzyżowa przeprowadzona pierwotnie na Railway (5/5 kryteriów) i Fly.io
(wybór #2). Po odrzuceniu obu z powodów kosztowych użytkownik wybrał Hetzner VPS z powodu
istniejącej infrastruktury i doświadczenia z platformą.*

### Adwokat diabła — Słabe strony

1. **Zero platform-level rollback** — Rollback = ręczne `docker compose pull` poprzedniego
   tagu + restart. Wymaga tagowania obrazów w GHCR (nie tylko `latest`) aby mieć poprzednie
   wersje dostępne.
2. **Postgres backup jest obowiązkiem użytkownika** — Brak automatycznych backupów. Utrata
   danych przy awarii dysku bez skonfigurowanego `pg_dump` + cron.
3. **OS patches wymagają ręcznej interwencji** — `apt upgrade` nie dzieje się automatycznie.
   Kernel exploity lub podatności Django wymagają aktywnego monitorowania i patching.
4. **Caddy + Cloudflare Origin Cert wymaga ręcznego odnawiania** — Origin Certificate jest
   ważny 15 lat (Cloudflare default), ale wymaga ręcznej aktualizacji gdy wygaśnie.
5. **Jeden VM = single point of failure** — Awaria CX22 = downtime. Brak HA na MVP jest
   akceptowalny, ale wymaga świadomej decyzji.

### Pre-Mortem — Jak to mogło się nie udać

Cztery miesiące po wdrożeniu na Hetzner projekt miał nieplanowany downtime. Początkowo
wszystko działało: Docker Compose uruchomił się bez problemów, Caddy + Cloudflare obsługiwały
HTTPS, deploy przez GitHub Actions był przewidywalny. Problem zaczął się w trzecim miesiącu
gdy dysk VM zaczął się zapełniać — stare obrazy Docker akumulowały się bez automatycznego
czyszczenia (`docker system prune`). W nocy z czwartku na piątek dysk osiągnął 100%,
PostgreSQL nie mógł zapisać WAL, baza danych się "zawiesiła". Odzyskanie zajęło 2 godziny
debugowania o 2 w nocy. Następnym problemem było odkrycie miesiąc później, że `pg_dump`
w cronie od 3 tygodni failował cicho (zmieniło się hasło Postgres w `.env.production` po
rotacji, ale nie w skrypcie backupu). Brak backupów przez 3 tygodnie pozostał niezauważony.

### Nieznane niewiadome

1. **Docker image cleanup nie jest automatyczny** — Bez `docker image prune` lub `watchtower`
   stare warstwy obrazów akumulują się na dysku. CX22 ma 80GB dysku — przy dużych obrazach
   możliwe zapełnienie w ciągu tygodni.
2. **Cloudflare "Full strict" wymaga certyfikatu na serwerze** — Bez Origin Certificate Caddy
   musi użyć self-signed, co w trybie "Full strict" zakończy się błędem 526. Wiele tutoriali
   opisuje "Full" (nie strict), które akceptuje self-signed — to różne tryby.
3. **`docker compose run --rm web python manage.py migrate` w CI/CD wymaga dostępu do bazy** —
   Jeśli kontener `db` nie jest uruchomiony lub healthcheck nie przeszedł, migracje failują
   w GitHub Actions SSH step. Wymaga `depends_on` z `condition: service_healthy`.
4. **GitHub Container Registry wymaga `packages: write` permission** — Bez jawnego ustawienia
   w workflow GITHUB_TOKEN nie ma uprawnień do pushowania do GHCR. Częsty błąd przy
   pierwszym setupie.

## Historia operacyjna

- **Wdrożenia podglądowe**: Brak natywnych preview URLs. Staging to osobny Docker Compose
  stack na tym samym VM z innym portem, lub osobna subdomená Cloudflare.

- **Sekrety**: Przechowywane w `/opt/ws-info-share/.env.production` na serwerze (chmod 600).
  Rotacja: edytuj plik + `docker compose up -d web` (restart kontenera). Nigdy nie
  commituj `.env.production` do repo — `.dockerignore` i `.gitignore` muszą go wykluczać.

- **Wycofywanie**: Taguj obrazy w GHCR: `ghcr.io/vojtech73/ws-info-share:<git-sha>` + `latest`.
  Rollback: SSH na serwer → `docker compose pull web` (z explicit tag) → `docker compose up -d`.
  Brak atomowego rollback — downtime możliwy jeśli nowy obraz jest już uruchomiony.

- **Zatwierdzanie**: Operacje tylko dla człowieka: usunięcie bazy / wolumenu Docker
  (`docker volume rm`), rotacja klucza SSH, zmiany firewalla Hetzner. Agent może: deploy
  przez GitHub Actions, sprawdzenie logów przez SSH.

- **Logi**: `docker compose logs web --tail=100` lub `docker compose logs web -f` (streaming).
  Brak centralnego log aggregatora w MVP — logi tylko na serwerze.

## Rejestr ryzyka

| Ryzyko | Źródło | Prawdopodobieństwo | Wpływ | Łagodzenie |
|---|---|---|---|---|
| Dysk VM zapełniony przez stare obrazy Docker | Pre-mortem | Średnie | Wysoki (DB downtime) | Dodaj `docker system prune -f` do monthly cron lub uruchamiaj po każdym deploy |
| Brak backupu Postgres → utrata danych | Adwokat diabła | Średnie | Wysoki (dane nieodwracalne) | Skonfiguruj `pg_dump` + cron po MVP; na start przynajmniej ręczny dump przed każdą migracją |
| Cloudflare Full strict + brak Origin Cert = error 526 | Nieznane niewiadome | Wysokie (przy setupie) | Średni (app niedostępna) | Wygeneruj Origin Certificate PRZED przestawieniem na Full strict; testuj z "Full" najpierw |
| GitHub Actions bez `packages: write` → GHCR push fail | Nieznane niewiadome | Średnie (przy setupie) | Średni (CI/CD broken) | Jawnie dodaj `permissions: packages: write` do job `build-push` |
| Migracje w CI/CD failują gdy `db` nie gotowy | Nieznane niewiadome | Niskie | Średni (deploy blocked) | Użyj `depends_on: condition: service_healthy` w docker-compose.yml |
| Single point of failure (jeden VM) | Adwokat diabła | Niskie | Wysoki (total downtime) | Akceptowalne dla MVP; plan HA po potwierdzeniu product-market fit |
| SECRET_KEY hardcoded w ustawieniach lokalnych → przypadkowy commit | Wynik badań | Niskie | Wysoki (bezpieczeństwo) | `config/settings.py` czyta SECRET_KEY z env; lokalne `.env` w `.gitignore` |

## Rozpoczęcie pracy

Konkretne pierwsze kroki dla Python/Django 6.0.5 + uv na Hetzner + Docker Compose:

1. **Na serwerze (jednorazowo)**: zainstaluj Docker, utwórz `/opt/ws-info-share/`,
   skopiuj `docker-compose.yml`, utwórz `.env.production` z `SECRET_KEY`, `DATABASE_URL`,
   `ALLOWED_HOSTS`.

2. **Cloudflare Origin Certificate**: wygeneruj w Cloudflare Dashboard → SSL/TLS → Origin Server,
   zapisz na serwerze jako `/etc/caddy/certs/origin.pem` i `origin.key`.

3. **GitHub Actions secrets**: dodaj `HETZNER_HOST`, `HETZNER_USER`, `HETZNER_SSH_KEY`
   do repozytorium (Settings → Secrets).

4. **Pierwsze uruchomienie**:
   ```bash
   cd /opt/ws-info-share
   docker compose pull
   docker compose up -d
   docker compose run --rm web python manage.py migrate
   docker compose logs web
   ```

## Poza zakresem

W niniejszych badaniach nie oceniano następujących kwestii:
- Konfiguracja obrazu Docker (poza wskazaniem błędnego wsgi path w auto-generated Dockerfile)
- Konfiguracja potoku CI/CD (GitHub Actions dla auto-deploy)
- Architektura na skalę produkcyjną (multi-region HA, disaster recovery)
