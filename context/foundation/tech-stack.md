---
starter_id: 10x-astro-starter
package_manager: npm
project_name: ws-info-share
hints:
  language_family: js
  team_size: solo
  deployment_target: cloudflare-pages
  ci_provider: github-actions
  ci_default_flow: auto-deploy-on-merge
  bootstrapper_confidence: first-class
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

wsInfoShare is an after-hours solo project — a multi-tenant B2B web platform with
role-specific panels (Platform Admin, Editor, Recipient) and a public-facing card
view accessed via QR code scans. The PRD signals large user scale, auth as the only
technology-forcing feature (FR-004), and a TypeScript + web product shape. `10x-astro-starter`
is the recommended default for `(web, js)`, clears all four agent-friendly gates
(typed, convention-based, popular in training data, well-documented), and ships auth,
PostgreSQL, and edge deployment in one integrated package. Supabase Auth satisfies
the external authentication service requirement from FR-004 out of the box. Cloudflare
Pages edge deployment directly addresses the ≤2 s public card view NFR — QR-scanned
pages load from the edge node closest to the end user. GitHub Actions with
auto-deploy-on-merge matches the solo after-hours workflow. Bootstrapper confidence
is first-class; scaffolding is expected to work without manual intervention.
