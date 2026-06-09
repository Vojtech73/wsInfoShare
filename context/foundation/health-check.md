---
project: ws-info-share
checked_at: 2026-06-05T00:00:00Z
health_status: needs-attention
context_type: brownfield
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
  high: 1
  moderate: 0
  low: 0
test_runner_detected: true
ci_provider: null
recommended_fixes: 4
---

## Dependency Health

### Lockfile

```
Status:          present (uv.lock)
Package manager: uv
```

`uv.lock` obecny — wszystkie wersje zależności zamrożone, kompilacje reprodukowalne.

### Security Audit

```
Tool:    pip-audit 2.10.0 (uv run pip-audit --format json)
Summary: 0 CRITICAL, 1 HIGH, 0 MODERATE, 0 LOW
Direct vs transitive: nie rozróżniane przez pip-audit w tym wywołaniu
```

#### HIGH findings

- **pip** 26.1.1 — PYSEC-2026-196 (CVE-2026-8643): pip traktuje `console_scripts` i
  `gui_scripts` jako ścieżki zamiast nazw plików, nie sanityzując bezwzględnej ścieżki
  instalacji — prowadzi do instalowania entry pointów poza katalogiem instalacji (path
  traversal przy instalowaniu złośliwego pakietu). Fix: zaktualizuj do pip 26.1.2.

> Uwaga kontekstowa: podatność dotyczy narzędzia `pip` wbudowanego w `.venv`, nie
> zależności produkcyjnych. Ryzyko dotyczy środowiska deweloperskiego podczas
> instalowania pakietów. Django, asgiref i inne zależności produkcyjne są czyste.

### Outdated Dependencies

```
Packages with major version gaps: 0
```

Wszystkie zależności bezpośrednie i przechodnie aktualne. Brak pakietów z lukami
w wersjach głównych.

---

## Test Suite

```
Test runner:    pytest 9.0.3
Tests found:    0 (runner działa, brak plików testowych)
Test execution: runner uruchomiony pomyślnie (uv run python -m pytest --collect-only)
```

```
Configuration: pyproject.toml [tool.pytest.ini_options]
Framework:     pytest 9.0.3 + pytest-django 4.12.0
```

Runner skonfigurowany i działa. Zarejestrowany w `[dependency-groups] dev`.
`DJANGO_SETTINGS_MODULE = "config.settings"` ustawiony poprawnie.
Brak plików testowych — agent może uruchamiać testy, ale nie ma jeszcze nic do
weryfikacji. Pierwszym krokiem implementacji powinno być napisanie choćby jednego
testu smoke-test.

---

## CI/CD

```
Provider:      not detected
Configuration: not found
```

Brak katalogu `.github/workflows/`, brak `.gitlab-ci.yml` ani innych plików CI.

```
ℹ Brak konfiguracji CI/CD. Skonfigurujesz to w lekcji dotyczącej infrastruktury
  i wdrożenia. Na razie lokalny runner testów wystarczy do współpracy z agentem.
```

| Stage      | Status | Notes                                       |
|------------|--------|---------------------------------------------|
| Lint       | ✗      | nie skonfigurowano                          |
| Test       | ✗      | nie skonfigurowano (runner lokalny działa)  |
| Build      | ✗      | nie skonfigurowano                          |
| Type check | ✗      | nie skonfigurowano                          |
| Security   | ✗      | nie skonfigurowano                          |

---

## Configuration

### High severity

Brak wysokich luk konfiguracyjnych.

### Medium severity

- **Brak lintowania i formatowania kodu** — `ruff` (linting + formatting) ani żaden
  inny narzędzie kodu nie jest skonfigurowane w `pyproject.toml`. Agent generuje kod
  niespójny stylistycznie bez egzekwowanego formatera. Fix: `uv add --dev ruff` +
  dodaj `[tool.ruff]` do `pyproject.toml`.

- **Brak sprawdzania typów** — `mypy` lub `pyright` nie są skonfigurowane. Django jest
  dynamicznie typowany (`typed: false` w rejestrze starterów), ale projekt może
  kompensować przez ścisłe typowanie w logice biznesowej. Bez CI wymuszającego mypy
  agent nie sygnalizuje błędów typów. Fix: `uv add --dev mypy django-stubs` +
  dodaj `[tool.mypy]` do `pyproject.toml`.

### Low severity

- **.editorconfig nieobecny** — brak spójnego formatowania między edytorami (indent,
  end-of-line). Fix: utwórz `.editorconfig` z `indent_size=4`, `end_of_line=lf`.

---

## Stack Assessment Cross-Reference

```
Brak pliku context/foundation/stack-assessment.md.
Uruchom /10x-stack-assess dla analizy bramek jakości stosu.
```

---

## Recommended Fixes

### Fix before agent work (Category A)

### 1. Zaktualizuj pip do 26.1.2 (podatność path traversal)

**Impact**: pip 26.1.1 zawiera podatność path traversal (PYSEC-2026-196) — złośliwy
pakiet może zapisywać pliki poza katalogiem instalacji podczas `pip install`. Ryzyko
dotyczy środowiska deweloperskiego.
**Severity**: high
**Effort**: quick (< 5 min)
**Fix**:

```bash
uv pip install --upgrade pip
```

Zweryfikuj po aktualizacji:

```bash
uv run pip-audit --format json
```

---

### 2. Dodaj ruff — linting i formatowanie

**Impact**: Bez formatera agent generuje kod niespójny stylistycznie (wcięcia, długość
linii, kolejność importów). Ruff jest najszybszym wyborem dla projektów Django/Python —
zastępuje black, isort, flake8 jedną zależnością.
**Severity**: medium
**Effort**: moderate (15–30 min)
**Fix**:

```bash
uv add --dev ruff
```

Dodaj do `pyproject.toml`:

```toml
[tool.ruff]
target-version = "py313"
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "UP"]
```

Uruchom formatowanie:

```bash
uv run ruff format .
uv run ruff check . --fix
```

---

### 3. Dodaj mypy — sprawdzanie typów

**Impact**: Django jest dynamicznie typowany, ale logika biznesowa projektu (unikalne
URL-e, izolacja tenantów) korzysta na statycznych typach. Bez mypy agent generuje
funkcje bez adnotacji typów, co utrudnia późniejszy refactoring.
**Severity**: medium
**Effort**: moderate (15–30 min)
**Fix**:

```bash
uv add --dev mypy django-stubs
```

Dodaj do `pyproject.toml`:

```toml
[tool.mypy]
python_version = "3.13"
strict = true
plugins = ["mypy_django_plugin.main"]

[tool.django-stubs]
django_settings_module = "config.settings"
```

Uruchom sprawdzenie:

```bash
uv run mypy .
```

---

### 4. Dodaj .editorconfig

**Impact**: Brak spójnego formatowania między edytorami — drobna niedogodność, ale
`ruff` + `.editorconfig` razem eliminują wszystkie stylistyczne rozbieżności.
**Severity**: low
**Effort**: quick (< 5 min)
**Fix**: Utwórz `.editorconfig` w katalogu głównym:

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
insert_final_newline = true
trim_trailing_whitespace = true

[*.py]
indent_size = 4

[*.{yml,yaml,json,toml}]
indent_size = 2
```

---

### Addressed in upcoming lessons (Category B)

### Brak CI/CD pipeline

**Lesson**: [Sprint Zero z Agentem: infrastruktura, walking skeleton i pierwszy deploy (M1L5)](https://platforma.przeprogramowani.pl/external/10xdevs-3/m1-l5)
**What you'll do there**: Konfiguracja GitHub Actions z etapami lint (ruff), test
(pytest), type-check (mypy) i auto-deploy do Fly.io.

### Brak konfiguracji wdrożenia

**Lesson**: [Sprint Zero z Agentem: infrastruktura, walking skeleton i pierwszy deploy (M1L5)](https://platforma.przeprogramowani.pl/external/10xdevs-3/m1-l5)
**What you'll do there**: Konfiguracja `fly.toml`, `Dockerfile` (Django + Gunicorn)
i zmiennych środowiskowych w Fly.io. Cel wdrożenia zmieniony na Fly.io (poprzedni
raport odwoływał się do Hetzner VPS — `tech-stack.md` zaktualizowany w tej sesji).

---

## Summary

```
Health status: needs-attention
```

Projekt poczynił znaczące postępy od ostatniej kontroli (2026-05-28): pytest
skonfigurowany i działa, pip-audit zarejestrowany jako dev dependency, `.env.example`
obecny, `.gitignore` oczyszczony z pozostałości Astro, AGENTS.md i CLAUDE.md obecne.
Główne luki to podatność HIGH w `pip` (path traversal, szybka poprawka — upgrade do
26.1.2) oraz brak lintowania i sprawdzania typów, co obniża jakość kodu generowanego
przez agenta. Runner testów działa, ale nie ma jeszcze żadnych testów do uruchomienia.

Next step: Wykonaj poprawki Kategorii A w kolejności (szacowany łączny czas: ~45 min),
napisz pierwszy test smoke-test, a następnie przejdź do lekcji infrastrukturalnej
(M1L5) w celu konfiguracji CI/CD i wdrożenia na Fly.io.
