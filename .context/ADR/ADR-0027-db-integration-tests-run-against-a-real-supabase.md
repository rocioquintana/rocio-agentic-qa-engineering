# ADR-0027 — The system under test proves isolation with DB-integration tests against a real Supabase

- **Status:** Accepted
- **Date:** 2026-11-10
- **Deciders:** QA lead of this project, confirmed at the project-discovery Phase 2 checkpoint on 2026-11-10. Discovered in the target repo, not decided there.
- **Tags:** isolation, fixtures, test-data, auth-in-tests, database
- **Supersedes:** —
- **Superseded by:** —

---

## Context

Discovered by `/project-discovery` Phase 2 while reading the target repo `upex-bunkai-tms`. Nobody decided this in that session; it is how the target's own tests work, and QA suites in this repo must live with it.

The target puts authorization in Postgres (row-level security and `SECURITY DEFINER` functions), so unit tests with mocks cannot prove tenant isolation. The target has about 45 `*-isolation.test.ts` files and some `*.db.test.ts` files that hit a real Supabase project. Examples: `lib/workspaces/member-removal-isolation.test.ts`, `lib/api/rls-parity.test.ts`, `lib/auth/rate-limit.db.test.ts`. They are skipped with a warning when the environment is absent. They sign in throwaway users through the real password path, tag fixtures with a prefix and assert zero leftovers in `afterAll`. A table `public.bunkai_test_failpoints` (migration `0114_workspace_member_removal.sql`) lets a test force a failure after every other write to prove rollback. The target's ADR-0012 (Proposed) requires a DB-integration test per definer function; no ADR records the isolation, fixture or failpoint approach itself.

## Decision

QA treats these as the invariants of the target's test architecture:

- Isolation and authorization claims are proven against a real Supabase, never against mocks.
- Test data is created per test run with a unique prefix, and the test removes it and asserts nothing is left.
- Users in tests are real throwaway users that sign in through the production path.
- Rollback behavior is proven through the failpoints table, a test seam that lives inside the production schema.


## Consequences

- **Positive:** authorization bugs that mocks hide (RLS gaps, definer functions without an actor bind) are caught.
- **Negative / trade-offs:** tests need live credentials and a reachable database, so they are skipped silently in any environment without them; fixtures can leak when a run is killed; the failpoints table ships in the production schema.
- **Neutral / follow-ups:** this repo's Playwright and API suites must follow the same prefix-and-cleanup discipline when they create data on staging; confirm with the team whether the staging database is shared.

## Alternatives considered

- **Mock the database client everywhere** — rejected by the target for authorization claims: it cannot show that an RLS policy holds.
- **Per-run ephemeral database** — not used by the target; no evidence it was weighed.

## References

- Target: `lib/workspaces/member-removal-isolation.test.ts`, `lib/api/rls-parity.test.ts`, `supabase/migrations/0114_workspace_member_removal.sql`, `.context/ADR/ADR-0012-rpc-authorization-invariant.md` (target repo)
- `infra-context` map: `bun run context:map infra-context --section nfr-security`
