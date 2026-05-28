---
project: ws-info-share
checked_at: 2026-05-28T00:00:00Z
health_status: needs-attention
context_type: greenfield
language_family: python
stack_assessment_available: false
checks_run:
  - lockfile
  - dependency_audit
  - outdated_deps
  - test_runner
  - ci_cd
  - configuration
audit_findings:
  critical: 0
  high: 0
  moderate: 0
  low: 0
test_runner_detected: false
ci_provider: null
recommended_fixes: 4
---

## Dependency Health

### Lockfile

```
Status:          present (uv.lock)
Package manager: uv
```

`uv.lock` wygenerowany automatycznie podczas bootstrapu. Wszystkie wersje zależności zamrożone — kompilacje są reprodukowalne.

### Security Audit

```
Tool:    pip-audit 2.10.0 (uv run pip-audit --format json)
Summary: 0 CRITICAL, 0 HIGH, 0 MODERATE, 0 LOW
Direct vs transitive: nie rozróżniane przez pip-audit
```

Brak znanych podatności. Pakiety audytowane: asgiref 3.11.1, django 6.0.5, sqlparse 0.5.5.

> Uwaga: `pip-audit` zainstalowany tymczasowo przez `uv pip install` podczas bootstrapu — nie jest zarejestrowany w `pyproject.toml` jako zależność deweloperska. Na innej maszynie `uv run pip-audit` nie zadziała. Patrz Recommended Fixes #3.

### Outdated Dependencies

```
Packages with major version gaps: 0
```

Drobne aktualizacje patch (nie wymagają działania):

- **idna**: 3.16 → 3.17 (patch)
- **platformdirs**: 4.9.6 → 4.10.0 (patch)

---

## Test Suite

```
Test runner:    nie wykryto
Tests found:    nie dotyczy
Test execution: nie podjęto próby
```

Brak konfiguracji pytest w `pyproject.toml`, brak `pytest.ini`, `tox.ini`, `setup.cfg`. Projekt nie ma żadnych testów ani zdefiniowanego runnera.

```
⚠ Brak test runnera. Agent nie może weryfikować własnych zmian.
Zalecane: pytest — najpopularniejszy runner dla projektów Django.
Setup: uv add --dev pytest pytest-django
```

---

## CI/CD

```
Provider:      nie wykryto
Configuration: nie znaleziono
```

Brak katalogu `.github/workflows/`, brak `.gitlab-ci.yml` ani innych plików CI.

```
ℹ Brak konfiguracji CI/CD. Skonfigurujesz to w lekcji dotyczącej infrastruktury
  i wdrożenia. Na razie lokalny runner testów wystarczy do współpracy z agentem.
```

---

## Configuration

### High severity

- **pytest (dev dependency)** — brak test runnera uniemożliwia agentowi weryfikację własnych zmian. Fix: `uv add --dev pytest pytest-django` + konfiguracja w `pyproject.toml`.

### Medium severity

- **.gitignore zawiera pozostałości Astro/Node.js** — obecne są wpisy dla `node_modules/`, `.astro/`, `wrangler/`, `.dev.vars`, `npm-debug.log*` itp. z poprzedniego stacku. Nie blokują pracy, ale zaśmiecają konfigurację. Fix: usuń zbędne sekcje JS/Cloudflare z `.gitignore`.

- **pip-audit nie jest zarejestrowany jako dev dependency** — zainstalowany tymczasowo, nie pojawia się w `pyproject.toml`. Fix: `uv add --dev pip-audit`.

### Low severity

- **.env.example nieobecny** — projekt Django wymaga zmiennych środowiskowych (`SECRET_KEY`, `DATABASE_URL`, `REDIS_URL`). Brak szablonu zmiennych to potencjalna pułapka dla nowych deweloperów i agentów. Fix: utwórz `.env.example` z pustymi wartościami dla wymaganych zmiennych.

- **.editorconfig nieobecny** — brak spójnego formatowania między edytorami. Fix: dodaj `.editorconfig` z ustawieniami Python (indent_size=4, end_of_line=lf).

---

## Stack Assessment Cross-Reference

```
Brak pliku context/foundation/stack-assessment.md.
Uruchom /10x-stack-assess dla analizy bramek jakości stosu.
```

---

## Recommended Fixes

### Fix before agent work (Category A)

### 1. Dodaj pytest i skonfiguruj test runner

**Impact**: Bez test runnera agent nie może weryfikować własnych zmian. To najważniejsza luka dla efektywnej współpracy z agentem.
**Severity**: high
**Effort**: quick (< 5 min)
**Fix**:

```bash
source ~/.local/bin/env
uv add --dev pytest pytest-django
```

Dodaj do `pyproject.toml`:

```toml
[tool.pytest.ini_options]
DJANGO_SETTINGS_MODULE = "config.settings"
python_files = ["tests.py", "test_*.py", "*_tests.py"]
```

### 2. Oczyść .gitignore z pozostałości Astro

**Impact**: Zaśmiecona konfiguracja może dezorientować agenta przy analizie projektu — wpisy dla `node_modules/`, `.astro/`, Cloudflare nie mają zastosowania w projekcie Django/Python.
**Severity**: medium
**Effort**: quick (< 5 min)
**Fix**: Usuń z `.gitignore` sekcje: `# build output` (`.astro/`), `# dependencies` (`node_modules/`), `# logs` (npm/yarn/pnpm), `# cloudflare` (`.dev.vars`, `.wrangler/`), powiel wpis `dist/` → zostaw jeden.

### 3. Zarejestruj pip-audit jako dev dependency

**Impact**: Bez wpisu w `pyproject.toml` audyt nie zadziała na nowej maszynie (`uv sync` go nie zainstaluje).
**Severity**: medium
**Effort**: quick (< 5 min)
**Fix**:

```bash
uv add --dev pip-audit
```

### 4. Dodaj .env.example

**Impact**: Agent i nowi deweloperzy nie wiedzą jakich zmiennych środowiskowych wymaga aplikacja. Django bez `SECRET_KEY` nie wystartuje.
**Severity**: low
**Effort**: quick (< 5 min)
**Fix**: Utwórz `.env.example`:

```bash
SECRET_KEY=your-secret-key-here
DEBUG=True
DATABASE_URL=postgres://user:password@localhost:5432/ws_info_share
REDIS_URL=redis://localhost:6379/0
ALLOWED_HOSTS=localhost,127.0.0.1
```

---

### Addressed in upcoming lessons (Category B)

### Brak CI/CD pipeline

**Lesson**: Sprint Zero z Agentem: infrastruktura, walking skeleton i pierwszy deploy (M1L5)
**What you'll do there**: Skonfigurowanie GitHub Actions z etapami lint, test, build i auto-deploy do środowiska docelowego.

### Brak AGENTS.md / kontekstu agenta

**Lesson**: Agent Onboarding: Agents.md, AI Rules i feedback loops (M1L4)
**What you'll do there**: Wygenerowanie AGENTS.md z konwencjami projektu, strukturą Django i zasadami pracy z agentem w tym stacku.

### Brak konfiguracji wdrożenia (Docker Compose, Nginx)

**Lesson**: Sprint Zero z Agentem: infrastruktura, walking skeleton i pierwszy deploy (M1L5)
**What you'll do there**: Konfiguracja Docker Compose dla Django + PostgreSQL + Redis + Nginx pod Hetzner VPS.

---

## Summary

```
Health status: needs-attention
```

Projekt jest świeżo po bootstrapie — zależności czyste (0 podatności), lockfile obecny, Django 6.0.5 zainstalowany. Główna luka to brak test runnera: bez pytest agent nie może weryfikować własnych zmian, co znacząco obniża niezawodność współpracy. Pozostałe problemy to porządkowe: .gitignore z pozostałościami Astro oraz brakujące .env.example. Brak CI/CD i AGENTS.md to oczekiwane luki na tym etapie — zostaną uzupełnione w kolejnych lekcjach.

Next step: Wykonaj poprawki Kategorii A (szacowany czas: ~15 minut), następnie przejdź do onboardingu agenta (/10x-agents-md).
