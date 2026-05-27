---
starter_id: django
package_manager: uv
project_name: ws-info-share
hints:
  language_family: python
  team_size: solo
  deployment_target: self-host
  ci_provider: github-actions
  ci_default_flow: auto-deploy-on-merge
  bootstrapper_confidence: verified
  path_taken: standard
  quality_override: false
  self_check_answers: null
  has_auth: true
  has_payments: false
  has_realtime: false
  has_ai: false
  has_background_jobs: false
---

## Why this stack

Solo developer building a multi-tenant B2B web platform after-hours on a single
Hetzner VPS. Django is the recommended default for `(web, python)` and clears all
four agent-friendly gates. Authentication uses django-allauth with django-allauth-2fa
for MFA — zero external containers, native Django ORM integration, covers email/password
and invitation flows required by the PRD. Redis is present as Django's cache and
session backend; application-level Celery is excluded from MVP to minimize operational
surface but the settings skeleton will be Celery-ready so the worker can be added
without refactoring. Frontend is HTMX + Alpine.js + Tailwind CSS + Flowbite —
hypermedia-driven, no SPA build step, fits Django's server-render model. Public card
view (QR scan, no login) uses plain Django Templates for minimum load latency,
satisfying the ≤2s NFR. Multi-tenancy via shared schema with tenant_id on every model
and queryset-level filtering; PostgreSQL RLS deferred to v2 per PRD non-goals. All
services on one VM: Django (Gunicorn), PostgreSQL, Redis, Nginx reverse proxy.
