# Project Configuration

> Project: rocioq-qa-engineering (system under test: Bunkai, `upex-bunkai-tms`)
> Generated: 2026-11-10 by `/project-discovery` Phase 1 (read-only discovery of the target repo)

## Repositories

| Repository | URL | Branch | Purpose |
|------------|-----|--------|---------|
| upex-bunkai-tms (local clone: `../upex-bunkai-tms`) | https://github.com/upex-galaxy/upex-bunkai-tms | `staging` (remote default branch, verified via `origin/HEAD`); `main` is production | Full-stack web application (Next.js App Router + Supabase) |

Layout: single repo, not a monorepo. Pages, route handlers (`app/api/v1/**`, 111 `route.ts` files) and domain logic (`lib/`) live together. The schema is in `supabase/migrations/` (119 files, numbered `0001` to `0121`).

The target's root `README.md`, `CONTEXT.md` and `INSTALLER.md` are generic boilerplate docs and say nothing about the product. Product knowledge lives in the target's `.context/business/`, `.context/PRD/`, `.context/SRS/` and `.context/ADR/`.

## Tech Stack

### Frontend
- Framework: Next.js `^15` (App Router, `typedRoutes`), React `^19`
- Language: TypeScript `^5.9.3`, `strict: true`
- Styling: Tailwind `^3.4`, shadcn-style components (`components.json`), Radix primitives, `cmdk`, `sonner`, `lucide-react`
- State: no state library; React context (`components/providers/auth-context.tsx`, `components/chat/ChatProvider.tsx`, the workbench context)
- Other UI: `@tanstack/react-table`, `@dnd-kit/*`, `@monaco-editor/react`, `shiki`, `react-markdown` with `remark-gfm` and `rehype-sanitize`, `@scalar/api-reference-react` (API docs UI at `app/api/docs`)

### Backend
- Framework: Next.js route handlers, REST under `app/api/v1/**` (`route.ts` + `route.openapi.ts`)
- Language: TypeScript
- ORM: none; `@supabase/supabase-js ^2` and `@supabase/ssr`, plus SQL RPC functions and RLS
- Contract: `zod ^4` with `@asteasolutions/zod-to-openapi`; output `api/openapi.json` and `api/openapi-types.ts`
- Auth: Supabase Auth with cookie sessions (`middleware.ts`); email/password, magic link, OAuth `github` and `google`; Personal Access Tokens for API and CLI (`lib/api/pat.ts`)
- Integrations: Stripe (billing, webhook), Resend (email), Jira (in-app connection and import, `lib/jira/`)
- Crons (`vercel.json`): `/api/v1/admin/send-digest` daily 08:00, `/api/v1/admin/storage-cleanup` daily 03:00

### Database
- Type: PostgreSQL
- Provider: Supabase (Auth, Realtime, Storage)
- Access: DBHub MCP, read-only QA role, via `DBHUB_*` variable names (`dbhub.toml` in the target). Not reachable in this session.

### Infrastructure
- Cloud: Vercel (the target repo has `vercel.json`; Git-integration deploy is inferred, not verifiable from the repo)
- CI/CD: none in the repo (no `.github/`); local gates only (`.husky/pre-commit`, `commit-msg`, `pre-push`)
- Monitoring: none found in `package.json`
- Runtime and package manager: Bun (`bun.lock`, `bun test`); no Node or Bun version pinned

## Environments

| Environment | URL | Purpose | Access |
|-------------|-----|---------|--------|
| Local       | {{environments.local.web_url}} | Dev | Direct |
| Staging     | {{environments.staging.web_url}} | Pre-prod testing | Test user credentials in `.env` by name (`STAGING_USER_EMAIL`, `STAGING_USER_PASSWORD`) |
| Production  | https://upexbunkai.vercel.app (`webapp_domain` in this repo); `bunkai.io` is a planned domain; `environments.production.web_url` still null here | Live | Read-only |

The API lives on the same origin under `/api/v1` (health endpoint: `app/api/v1/health`).

## Tools and Access

- Issue tracker: Jira, resolved via `/acli`; host in `issue_tracker.atlassian_url` of `.agents/project.yaml`
- Project key: {{PROJECT_KEY}}
- Database: `[DB_TOOL]` resolves to the DBHub MCP (needs `DBHUB_*` values)
- API spec source: `public/openapi.json` in the target repo (OpenAPI 3.1, 110 paths, generated from zod by `bun run openapi:gen` there), served at `/api/openapi` with a Scalar UI at `/api/docs`. Feed this one to `bun run api:sync`. The copy at `api/openapi.json` is stale (78 paths).
- Env variable names the target reads (names only): Supabase (`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_JWT_SECRET`), app (`NEXT_PUBLIC_APP_URL`), Stripe (`STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `STRIPE_CLOUD_PRICE_ID`), Resend and cron (`RESEND_API_KEY`, `RESEND_DIGEST_FROM_EMAIL`, `CRON_SECRET`)
- Docs: the target's `.context/business/`, `.context/PRD/`, `.context/SRS/`, `.context/ADR/`, `DESIGN.md`, `docs/`

## Access Checklist

- [x] Repository read access (local clone)
- [ ] Database access (DBHub MCP failed to connect; `DBHUB_*` values not confirmed)
- [x] Issue tracker access (Phase 4: `acli` authenticated on the host in `issue_tracker.atlassian_url`; `BK-1` read OK; `bun run jira:check` exit 0, 252 OK, 0 missing; link types 10 OK)
- [ ] Staging environment reachable (not verified; no test user credentials confirmed)
- [ ] CI/CD visibility (the target has no CI: nothing to see)

## Discovery Gaps

- [ ] CI and deploy path: no `.github/` workflows. Confirm in the Vercel project settings whether Git integration deploys `staging` and `main`.
- [ ] Production `web_url` and `api_url`: `environments.production` is still null in `.agents/project.yaml`; the target's yaml gives `https://upexbunkai.vercel.app`. Fill it when the team confirms.
- [ ] Branch protection of the target: default branch is `staging` (verified); protection rules not checked.
- [ ] One Supabase project backs local, staging and production (target yaml claim): QA data on staging is production data. Confirm with the team.
- [ ] Env name drift: `.env.example` lists `SUPABASE_PUBLISHABLE_KEY` and `SUPABASE_SECRET_KEY`, while `lib/env.ts` validates `NEXT_PUBLIC_SUPABASE_ANON_KEY` and `SUPABASE_SERVICE_ROLE_KEY`. Check `.env.core.schema` and the deployed Vercel env to learn which names are real.
- [ ] Secret sweep: regex hits (paths only) in `lib/billing/invoices.test.ts`, `lib/billing/checkout.test.ts`, `app/api/v1/billing/webhook/route.test.ts`, `docs/setup/mcp/README.md`, `docs/architectures/supabase-nextjs/auth-tokens.md`. Likely fixtures or examples; a human must confirm.
- [ ] Monitoring and observability: none found in the repo. Ask the team.
- [ ] Jira host: the connection works on the host in `.agents/project.yaml`. The target's `/qa` page links the credentials Epic BK-29 on another host (`jira.upexgalaxy.com`) and its roadmap mentions an unresolved two-host choice. Confirm which host is canonical before any write. Not blocking.
- [ ] Phase 4 hierarchy: `.agents/jira-required.yaml` declares story, bug and epic as `sync: default`; defect, test plan, test execution, test set and precondition as `discovery`; improvement, tech story and tech debt as `optional`; test case as `never`. Confirm it matches the team's real hierarchy.
- [ ] Whether the target's test suite passes: no script was run (read-only discovery).
- [ ] Auth flow details (middleware logic, RLS policies, PAT scopes): Phase 2 and Phase 3.
