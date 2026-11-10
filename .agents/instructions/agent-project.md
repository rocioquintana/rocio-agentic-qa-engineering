---
id: project
title: 'Project-specific instructions'
load_when: 'anything specific to this project: its own conventions, guardrails, environments or vocabulary not covered by another section'
triggers: []
paths: []
---

# Project-specific instructions

> Project-owned overlay. The boilerplate delivers this file once and never overwrites it; every other file in this folder is synced from upstream. Write this project's own rules here, one heading per topic, instead of editing `AGENTS.md` or a synced section. Knowledge about the system under test belongs in a project context skill (`<aspect>-context`), not here.

Add `triggers:` regex sources to the frontmatter above when a rule here should be routed by keyword (the hook reads them), and keep every `NEVER` / `MUST` line reachable: cite `Rule #N`, binding: `/<skill>`, or enforced: `bun run <script>` (checked by `bun run instructions:check`).

## Project context skills

One row per skill this project authored: the `<aspect>-context` skills `project-context` mode `context-skill` creates, and any other skill the project added. The skill router in `agent-skills-and-mcps.md` is synced, so `bun sync` overwrites a row written there; this file is never overwritten. Add each skill's trigger phrases to `triggers:` above as regex sources too, so the hook routes this file when a prompt names them. `bun run instructions:check` fails a row whose `.agents/skills/<slug>/SKILL.md` does not exist.

| Skill | Trigger | Purpose |
|---|---|---|

## Project Assessment (Phase 1)

Assessment Date: 2026-11-10

### Testing Maturity: 3/4
- Current state: Good (broad unit and route-handler coverage; no E2E, no coverage measurement, no CI execution)
- Test files: 403 `*.test.*`, 0 `*.spec.*`
- Frameworks: `bun test` with `@happy-dom/global-registrator`
- Coverage: unknown (no coverage tooling found); no SQL-level tests (no `supabase/tests`)

### Documentation State: Partial
- README: yes, but generic boilerplate content (nothing about the product)
- API docs: yes (`api/openapi.json`, Scalar UI at `app/api/docs`)
- Architecture: yes (`.context/ADR/`, `docs/architectures/`, `.context/design/`); product docs in `.context/business/`, `.context/PRD/`, `.context/SRS/`
- Setup guide: partial (generic, under `docs/setup/`)

### Code Quality
- [x] ESLint: configured (`eslint.config.js`, antfu base)
- [x] Prettier: configured (json, yml, css, html only; TS goes through ESLint)
- [x] TypeScript: strict
- [x] Pre-commit hooks: configured (husky + lint-staged), bypassable locally

### CI/CD Maturity: None
- No `.github/` workflows; gates are local husky hooks only; deploy path is not verifiable from the repo

### Identified Risks

| Risk | Severity | Evidence (path only) | Mitigation |
|------|----------|----------------------|------------|
| No CI: tests, lint and types are not enforced server-side | HIGH | `.husky/pre-push` (local only; no `.github/`) | Recommend a CI workflow in the target repo; QA suites here run against staging meanwhile |
| Possible hardcoded secrets (regex hits, unconfirmed, likely fixtures) | HIGH | `lib/billing/invoices.test.ts`, `lib/billing/checkout.test.ts`, `app/api/v1/billing/webhook/route.test.ts`, `docs/setup/mcp/README.md`, `docs/architectures/supabase-nextjs/auth-tokens.md` | Human triage of the five paths |
| No E2E and no coverage measurement | MEDIUM | `package.json` | This repo's Playwright + KATA suites fill the E2E gap |
| `.env.example` names differ from the names the app validates | MEDIUM | `.env.example`, `lib/env.ts` | Align the example with `lib/env.ts` |
| Public cron route guarded by a shared `CRON_SECRET`, no app-level rate limiter (per the target's own roadmap) | MEDIUM | `.context/ADR/ADR-0017-system-cron-principal-class.md`, `.context/dev-roadmap.md` | Security-test the cron routes in the test plan |
| Version drift: Next 15 with `@next/eslint-plugin-next` 16; no Node or Bun pin | LOW | `package.json` | Pin versions, align the plugin |

### Phase Prioritization

- Phase 1: Normal -- product docs and 119 migrations give strong evidence
- Phase 2: Extended -- 111 route handlers, Supabase RLS, Stripe, Resend and Jira integrations to map
- Phase 3: Normal -- no CI to map; Vercel deploy and environments only
- Phase 4: Extended -- confirm the Jira host before any sync

### Blockers
- [ ] Secret-sweep hits need human triage before the discovery outputs are shared
- [ ] DBHub MCP and staging test credentials not confirmed (needed from `project-context` mode `data` and sprint testing)
