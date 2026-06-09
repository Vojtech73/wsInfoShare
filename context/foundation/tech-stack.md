---
starter_id: django
package_manager: uv
project_name: ws-info-share
hints:
  language_family: python
  team_size: solo
  deployment_target: fly
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
