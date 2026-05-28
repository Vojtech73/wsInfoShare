# uv — ściągawka

`uv` to menedżer pakietów i środowisk Python napisany w Rust (od Astral, twórców Ruff).
Zastępuje pip + virtualenv + pip-tools + pyenv. Działa 10–100x szybciej od pip.

## Środowisko wirtualne

```bash
# uv tworzy .venv automatycznie przy pierwszym `uv add`
# Ręczne utworzenie:
uv venv

# Aktywacja (opcjonalna — `uv run` nie wymaga aktywacji)
source .venv/bin/activate

# Deaktywacja
deactivate
```

## Zarządzanie paczkami

```bash
uv add django                  # instalacja + aktualizacja pyproject.toml i uv.lock
uv add "django>=5.0"           # z ograniczeniem wersji
uv add --dev pytest ruff       # zależności developerskie
uv remove django               # usunięcie paczki
uv sync                        # odtworzenie środowiska z uv.lock (odpowiednik npm install)
uv lock                        # regeneracja uv.lock bez instalacji
```

## Uruchamianie poleceń

```bash
uv run manage.py runserver     # uruchomienie w .venv bez aktywacji
uv run manage.py migrate
uv run manage.py createsuperuser
uv run pytest
uv run python skrypt.py
```

## Inicjalizacja projektu

```bash
uv init --no-readme --name moj-projekt   # nowy projekt (tworzy pyproject.toml)
uv python install 3.13                   # instalacja konkretnej wersji Pythona
```

## Wersje Pythona

```bash
uv python list                 # dostępne wersje
uv python install 3.12         # instalacja wersji
uv python pin 3.13             # przypięcie wersji w .python-version
```

## Onboarding (nowy deweloper / nowa maszyna)

```bash
git clone <repo>
cd <repo>
uv sync                        # odtwarza .venv i instaluje wszystko z uv.lock
uv run manage.py runserver     # gotowe
```

## Pliki generowane przez uv

| Plik | Opis | Git |
|---|---|---|
| `pyproject.toml` | zależności projektu | ✅ śledzony |
| `uv.lock` | zamrożone wersje wszystkich paczek | ✅ śledzony |
| `.venv/` | środowisko wirtualne | ❌ .gitignore |
| `.python-version` | wymagana wersja Pythona | ❌ .gitignore |

## PATH (WSL / Linux)

Po instalacji przez `curl` dodaj do `~/.bashrc` lub `~/.zshrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Lub jednorazowo w sesji:

```bash
source $HOME/.local/bin/env
```
