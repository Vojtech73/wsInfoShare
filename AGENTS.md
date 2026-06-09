# Repository Guidelines

ws-info-share is a multi-tenant B2B platform for information card distribution: Editor publishes cards → Recipient generates a QR link → public view without login. Stack: Django 6.0.5 + Python 3.13 + PostgreSQL (planned) + HTMX + Alpine.js + Tailwind CSS + Flowbite.

## Hard rules

- Run all Python commands via `uv run` — never bare `python` or `./manage.py`. Correct: `uv run manage.py runserver`. Wrong: `python manage.py runserver`.
- Never commit `.env`. Copy `.env.example` → `.env` and fill in real values. The `SECRET_KEY` in `@config/settings.py` is an insecure scaffold placeholder — override it via env in every environment.
- Never modify files under `context/`. It contains read-only project documentation (PRD, tech-stack decisions, health-check reports).
- The Django settings module is `config/`, not a directory named after the project. Entry points: `ROOT_URLCONF = "config.urls"`, `WSGI_APPLICATION = "config.wsgi.application"`.

## Project structure

`config/` — Django settings module (`settings.py`, `urls.py`, `wsgi.py`, `asgi.py`). `context/` — read-only project docs. `notes/` — developer notes. See `@pyproject.toml` for dependencies and pytest config, `@.env.example` for required env vars, `@context/foundation/tech-stack.md` for full stack rationale.

Django apps live at the root level (`cards/`, `tenants/`, `accounts/`), each following the standard Django layout: `models.py`, `views.py`, `urls.py`, `admin.py`, `tests.py`.

## Commands

- `uv sync` — restore `.venv` from `uv.lock` (first-time setup or after `git pull`)
- `uv run manage.py runserver` — dev server
- `uv run manage.py migrate` — apply migrations
- `uv run manage.py makemigrations` — generate migrations after model changes
- `uv run pytest` — full test suite
- `uv run pip-audit` — dependency security audit

## Multi-tenancy convention

Every domain model carries a `tenant` ForeignKey. All querysets must be filtered through a `TenantQuerySet` scoped to `request.tenant`. Never return cross-tenant data from any view or API endpoint.

## Testing

pytest + pytest-django. `DJANGO_SETTINGS_MODULE = "config.settings"` is pre-configured in `@pyproject.toml`. Place tests in `tests.py` within each Django app or as `test_*.py` files. Run a single test: `uv run pytest path/to/tests.py::test_name`.

## Commits

Conventional Commits observed in history: `feat:`, `fix:`, `chore:`, `docs:`. Messages in Polish or English — match the language of surrounding commits in the PR context.

## Configuration and secrets

Required env vars: `@.env.example`. Dev database is SQLite; PostgreSQL is the production target via `DATABASE_URL`. Redis planned for cache/sessions via `REDIS_URL`.
