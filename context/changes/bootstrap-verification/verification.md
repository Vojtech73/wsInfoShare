---
bootstrapped_at: 2026-06-05T00:00:00Z
starter_id: django
starter_name: Django
project_name: ws-info-share
language_family: python
package_manager: uv
cwd_strategy: native-cwd
bootstrapper_confidence: verified
phase_3_status: failed
audit_command: pip-audit
---

## Hand-off

Frontmatter z `context/foundation/tech-stack.md` (stan po aktualizacji 2026-06-05):

| Field | Value |
|---|---|
| starter_id | django |
| package_manager | uv |
| project_name | ws-info-share |
| language_family | python |
| bootstrapper_confidence | verified |
| path_taken | standard |
| deployment_target | fly |
| has_auth | true |
| has_payments | false |
| has_realtime | false |
| has_ai | false |
| has_background_jobs | false |

### Why this stack

Projekt wsInfoShare to wielotenantowa platforma B2B realizowana po godzinach w
Pythonie. Django jest rekomendowaną wartością domyślną dla `(saas, python)` i
spełnia trzy z czterech kryteriów przyjaznych agentowi (convention-based,
popular_in_training, well_documented; `typed: false` jest mitygowane przez
dyscyplinę typów w logice biznesowej). Auth jest w zakresie MVP (FR-004) —
wbudowany system auth Django jest bezpośrednim dopasowaniem. Model
wielotenantowy (rola przypisana do tenanta, nie do użytkownika) mapuje się
naturalnie na ORM Django z polem `tenant_id` na każdym modelu i filtrowaniem
na poziomie queryset. Publiczny widok Karty (FR-015) — otwierany przez skan
QR bez logowania — spełnia wymaganie NFR ≤2s przez standardowe Django
Templates bez kroku SPA. Frontend realizowany przez HTMX + Alpine.js +
Tailwind CSS + Flowbite — hipermedialne podejście bez kroku budowania SPA, w
pełni zintegrowane z Django Templates. Fly.io jako cel wdrożenia:
konteneryzowany deploy, managed PostgreSQL add-on, workflow CLI-first
(`flyctl`), hojny darmowy tier na MVP. CI na GitHub Actions z auto-deploy po
merge do main.

---

## Pre-scaffold verification

| Signal      | Value                                            | Severity | Notes                                                             |
|-------------|--------------------------------------------------|----------|-------------------------------------------------------------------|
| npm package | not run                                          | —        | nie dotyczy — starter nie-JS                                      |
| GitHub repo | not run                                          | —        | docs_url (`https://docs.djangoproject.com`) nie jest adresem GitHub |

Brak sygnału aktualności. Kontynuowano bez ostrzeżenia.

---

## Scaffold log

**Resolved invocation**: `uv run django-admin startproject . .`
**Strategy**: native-cwd
**Exit code**: 1
**Pre-flight files-to-touch**: `manage.py`, `config/__init__.py`, `config/settings.py`, `config/urls.py`, `config/wsgi.py`, `config/asgi.py`
**Stderr (last 20 lines)**:

```
CommandError: '.' is not a valid project name. Please make sure the name is a valid identifier.
```

**.bootstrap-scaffold**: nie dotyczy — strategia native-cwd nie tworzy katalogu tymczasowego.

**Przyczyna awarii**: strategia `native-cwd` podstawia `{name}=.` w szablonie
`django-admin startproject {name} .`, co daje `django-admin startproject . .`.
Polecenie `django-admin startproject` oczekuje ważnego identyfikatora Python jako
nazwy projektu — `.` jest nieprawidłowe.

**Kontekst**: projekt był już poprawnie zainicjowany w poprzednim uruchomieniu
bootstrappera (2026-05-27) poleceniem `uv run django-admin startproject config .`.
Bieżące uruchomienie było próbą ponownego bootstrapu po zmianie `deployment_target`
z `self-host` na `fly` w `tech-stack.md`. Pliki projektu są nienaruszone — cwd
nie uległo zmianie.

---

## Post-scaffold audit

**Audit not run**: scaffold halted at Step 2; no project to audit.

---

## Hints recorded but not acted on (v1)

| Hint                    | Value                  |
|-------------------------|------------------------|
| bootstrapper_confidence | verified               |
| quality_override        | false                  |
| path_taken              | standard               |
| self_check_answers      | null                   |
| team_size               | solo                   |
| deployment_target       | fly                    |
| ci_provider             | github-actions         |
| ci_default_flow         | auto-deploy-on-merge   |
| has_auth                | true                   |
| has_payments            | false                  |
| has_realtime            | false                  |
| has_ai                  | false                  |
| has_background_jobs     | false                  |

---

## Next steps

**Scaffold CLI zakończył się błędem — projekt jest jednak nienaruszony.**

Projekt Django (manage.py, config/, pyproject.toml, uv.lock) pochodzi z
poprzedniego uruchomienia bootstrappera (2026-05-27) i jest w pełni funkcjonalny.
Bieżące uruchomienie zakończyło się błędem z powodu ograniczenia szablonu
bootstrappera (native-cwd + django-admin startproject), nie z powodu błędu w
projekcie.

Zmiana `deployment_target: fly` w `tech-stack.md` to jedyna różnica względem
poprzedniego uruchomienia — nie wymaga ponownego uruchamiania scaffoldu.

Kolejne kroki:
- Uruchom `/10x-infra-research` (Plan Mode deploy) aby skonfigurować wdrożenie na Fly.io
- Zaimplementuj poprawki z `context/foundation/health-check.md` Kategoria A (pip upgrade, ruff, mypy)
