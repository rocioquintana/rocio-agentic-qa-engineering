# ADR-0028 — `data-testid` is the selector contract with the system under test

- **Status:** Accepted
- **Date:** 2026-11-10
- **Deciders:** QA lead of this project, confirmed at the project-discovery Phase 2 checkpoint on 2026-11-10. Discovered in the target repo, not decided there.
- **Tags:** selectors, ui, contract
- **Supersedes:** —
- **Superseded by:** —

---

## Context

Discovered by `/project-discovery` Phase 2 while reading the target repo `upex-bunkai-tms`. The target uses `data-testid` attributes broadly: about 1,161 occurrences across 139 `.tsx` files (counted with `rg`), in kebab-case with a domain prefix (for example `billing-plan-card`). No ADR or doc in the target states this as a rule; it is practice. The target has no E2E runner of its own, so this repo's Playwright suites will be the main consumer.

## Decision

UI tests in this repo locate elements by `data-testid` first, and treat a missing or renamed test id as a defect to raise with the target team, not as a reason to fall back to CSS or text selectors. Test ids follow the target's pattern: kebab-case, domain-prefixed.


## Consequences

- **Positive:** selectors survive restyling and copy changes; one rule for every UI test.
- **Negative / trade-offs:** the suite depends on the target team keeping ids stable, and there is no automated guard on their side (no E2E suite breaks when an id changes); elements without an id force a request to the target team.
- **Neutral / follow-ups:** agree with the target team how id changes are announced; check whether the `app/qa/` testability guide lists the ids.

## Alternatives considered

- **Role and accessible-name locators first** — good for accessibility checks, but copy changes break them, and the target has no localization bundle to anchor labels.
- **CSS and text selectors** — rejected: the most fragile against redesign.

## References

- Target: `components/**`, `app/**`, `app/qa/qa-config.ts` (target repo)
- `infra-context` map: `bun run context:map infra-context --section architecture`
