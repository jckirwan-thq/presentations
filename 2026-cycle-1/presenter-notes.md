# Cycle 01 · 2026 — Presenter Notes

Tradify Engineering Wrap · 23 Mar → 06 May 2026 · Jean-Claude Kirwan
Theme: *Back to the Future*. Tone: confident, playful, precise.

Keyboard during presentation: `←` `→` navigate · `B` brand toggle · `T` theme toggle.

---

## Slide 1 — Outatime · Masthead

- "Cycle 01, 2026. Twelve-year jump. Where we're going, we don't need a monolith."
- Land the dates (23 Mar → 06 May) and the ticket (TRA-22640) so the room knows the shape.
- Time circuits panel mirrors the dashboard from the film: **destination** is Cycle 02 · Exemplar; **present** is today's wrap; **last departure** was kickoff.
- Power draw "1.21 GW · a bolt of lightning" is the sight gag — don't explain it.
- Hit `B` to flip the brand from Tradify to Access mid-sentence — gets attention before any content.

---

## Slide 2 — Foundation · what got built

**This is the architecture story. Take your time.**

- Lead with **why this slide exists**: we did not "ship a feature." We replaced the React app entirely.
- The legacy `web-frontend` (the old monolithic React app, *not* AngularJS) stepped down. In its place: `tradify-frontend-platform` — a Module-Federation 2.0 host that contains every MFE.
- `@tradify/frontend-context` is the **portable API**. Every plugin reads from the same bus regardless of which shell it boots inside (Angular today, standalone tomorrow).
  - State slices, RTK Query injection, plugin registry, message bus — all in one package.
  - The point: **features only declare what's new** — auth, retry, cache, invalidation are inherited.
- Walk through the four sub-pillars under `/enquiries`:
  - **/me** — session, CSRF, identity. Bootstrap today via postMessage; tomorrow it's `GET /v2/api/v1/me`. Same contract, zero consumer churn. *Call this out — it's the contract that lets AngularJS retire without a Big Bang.*
  - **/analytics** — TRA-23263. Page-view telemetry, dev dashboard, parity with the legacy event stream.
  - **AngularJS message bridge** — postMessage choreography. Bootstrap, route sync, navigation interception, sidebar totals.
  - **/feature-adoption** — the carrot when an entitlement gate denies access. Reused inside `EntitlementGate`'s `fallback` (slide 4).
- `/enquiries` is the proof — it stands on all four sub-pillars in production.
- One-liner to close: "The host knows what route maps to what remote. It never knows what chunks exist."
- **If asked**: yes, the AngularJS shell still owns auth and the sidebar — that's intentional. Once `GET /me` lands server-side, /me flips data sources and nothing else changes.

---

## Slide 3 — Foundation · old dog, new tricks

**Story: the 2014 Angular shell didn't get rewritten — it learned new tricks.**

- Set the scene: production today is still the AngularJS shell. 444-line controllers. We are not rewriting it. We are **teaching it to broadcast**.
- Walk the four postMessage contracts left-to-right:
  - **PLATFORM_BOOTSTRAP** (TRA-23113) — Angular pushes session, CSRF, identity into React **proactively** at shell-load. No more "React boots, asks Angular, waits, retries." The white-screen killer (lands on slide 4).
  - **UPDATE_REACT_ROUTE** (TRA-22653) — when Angular changes the URL, React's router syncs deep-link.
  - **NAVIGATE_TO** (TRA-23319) — React asks Angular to navigate, with a **bypass-flag** so React-owned routes don't bounce back through Angular.
  - **PLATFORM_ENTITY_TOTALS_UPDATED** (TRA-23325) — sidebar count refresh after a mutation. Replaces a polling loop that ran every tab change.
- The diagram on the right: dashed iframe seam down the middle. Two columns. Four messages. That's the entire bridge.
- **The punchline**: this shell is going to retire. But before it does, it had to behave. And it does.
- **If asked about coupling**: the bridge is an explicit contract — every message has a ticket, a payload schema, a test. It's not a hack; it's a *negotiated handoff*.

---

## Slide 4 — Challenges · two leaks, both sealed

- **α · The six-second white screen**: cold-load `/enquiries` was 6.0s → 1.9s. That's a **−68%** drop.
  - Root causes: source aliases bleeding across plugin boundaries, eager-shared singletons leaking, a CDN-fetched manifest, polling on tab change.
  - Fixes: inlined manifest, preloaded remote entry, host-shell hoist of `/me`, killed polling, eager-shared follow caller.
  - Tickets: TRA-23095, TRA-23113, TRA-23293, TRA-23325, perf #59.
- **β · Doc Brown's padlock, ported**: AngularJS shipped `<upsell-padlock>`. Re-implementing it in an MFE that boots **before bootstrap arrives** surfaced two paradoxes — a contract the platform didn't have, and a loading state nobody wanted to see.
  - `EntitlementGate` resolves three states: granted · denied · loading.
  - `optimistic` mode renders children while bootstrap is in flight, swaps to fallback if denied. Trades a brief flash for happy-path speed (off for paid-only content).
  - Fallback is `FeatureAdoptionPanel` — heading, body, primary CTA, dismiss/remind. The same component upsells the user *and* serves as the empty-state when the feature exists but isn't set up.

---

## Slide 5 — Brand parity

- "One flux capacitor, two power sources." Hit `B`. Components flip from Tradify-blue to Access teal **at runtime**.
- Every component reads from token contracts under `[data-brand]`. No prop drilling. No theme provider hops. Just CSS variables.
- Tabs and Buttons shown side-by-side mirror the live Storybook. **No forking** — same components, different tokens.
- Tickets that did the work: TRA-23228 (Evo brand parity), TRA-23083 (brand theming consolidation, in review), TRA-22952 (tooltip styling contract), TRA-23163 (loading spinner standardisation).
- "The contract is the seam." That's the architectural claim — Access integration is a token swap, not a fork.

---

## Slide 6 — Agentic workflow

- Stood up `tradify-agent-skills` — a user-level repo of tech-lead agents and project skills, shared across every Tradify repo.
- Three columns:
  - **8 user-level tech leads** — frontend, backend, AngularJS, UX, e2e testing, DevOps, security, engineering-manager. "Each one a Doc Brown for its domain."
  - **7 repo-level specialists** — platform, frontend-context, ui-components, bundle perf, DevX, docs, code review. Live in `.claude/agents` close to the code.
  - **7 project skills** — Evo-shadcn, module federation, React 19 pinning, Tailwind v4 contracts, HTML5 + React, Storybook 10, cross-platform DX.
- **Why this matters**: the next migration phase is bigger than one engineer. The agents make Tradify-specific knowledge portable.

---

## Slide 7 — Developer experience

- "Hoverboard-grade developer loops." Four cards, four classes of tooling:
  - **Generators**: `generate:mfe`, `generate:evo-mfe`, `generate:context`, `generate:api-call`, `generate:icons`. Scaffold from one command.
  - **Targets & modes**: `dev`, `dev:qa`, `dev:mock`, `switch-api.mjs`, `ensure-env-local.mjs`. Same shell, many backends.
  - **Contracts & parity** (CI gates): `check:brand-parity`, `validate-shared-config`, `validate-manifest-consistency`, `validate-env-completeness`, `sync:singletons`.
  - **Dev-only views & agent mode**: `/analytics` dashboard, `/feature-adoption` playground, `--agent` non-interactive flag, `graph` script, Design-Sync DevX branch.
- The story isn't the list — it's that **every shape of work has a script**. Drift gets caught in CI before review.

---

## Slide 8 — Demo time

- Hand off to Nikita.
- Live walk-through of the migrated `/enquiries` surface, brand toggle in flight, agentic workflow in action.
- Don't talk over the demo. Let the running app do the talking.

---

## Slide 9 — The receipts

**This is the closer. Numbers are real. Lead with them.**

### The six metric cards (top half)

- **Unit test coverage · 95.1%** — 4,313 / 4,537 lines covered (combined report from `coverage/index.html`).
- **Projected · React-owned route · ~0.8s** — projected cold-load when `/enquiries` exits the iframe and becomes a React-owned route. No shell relay, no postMessage hop. *Today's 1.9s is the in-iframe number; 0.8s is what we get back when the bridge is no longer in the path.*
- **A11y · 36 / 49** components a11y-attributed (aria · roles · focus management).
- **/enquiries test files · 24** — 6,548 lines of spec.
- **CI · Turbo affected-graph · ~5×** speedup. Build & test only what changed.
- **Architecture · 4 · 2** — four new MFEs (`/me`, `/analytics`, `/FAP`, `/enquiries`), two brand surfaces (Tradify, Access).

### The diff panel (bottom half) — **the architecture story in 30 seconds**

- **Left (2014)** — `EnquiriesCtrl.js`. One controller, 444 lines. State, API, formatting, validation, navigation, DOM — all in one place. Every feature reinvents auth, retry, cache, invalidation. *That's the god-controller pattern.*
- **Right (2026)** — `enquiriesApi.ts`. The feature owns its slice. `baseApi.injectEndpoints({ list, save, remove })`. Auth, retry, cache, invalidation are solved **once** in `baseApi` and inherited.
- **Foot line** — say it out loud: "1 controller, 444 lines → N features, each owning ~80 lines. Auth/retry/cache solved once in `baseApi`. Adding a feature is `injectEndpoints`, not editing a god-controller."
- **The closer**: this is the pattern shift the cycle proved. Every future surface will follow it. The almanac doesn't lie.

### If asked

- "Why 95.1% and not 100%?" — Branches at 86%; we ratchet up branch coverage cycle-by-cycle. Lines is the headline number because lines are what reviewers read in PRs.
- "Why ~0.8s and not measured?" — In-iframe LCP is measured (1.9s). The 0.8s is projected from the same trace minus the postMessage relay and shell hoist. We'll measure once `/enquiries` ships out-of-iframe.
- "What's the AngularJS retirement plan?" — `/me` flips its data source from postMessage to `GET /v2/api/v1/me`. The shell retires. Every plugin else stays the same.

---

## Closing line (off-slide)

> "Roads? Where we're going, we don't need roads."
