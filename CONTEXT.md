# CONTEXT.md — Working Handoff

**Project:** gt-sandbox (product name TBD — see §10) · **Version:** v0.0.1 · **Updated:** 2026-07-13

This is the agent working file: the build plan, decisions ledger, verification protocol, and session log. It is deliberately separate from `README.md` (public-facing). README never gains agent instructions; CONTEXT never becomes marketing.

**Status:** Planning complete and owner-reviewed (2026-07-13): stack is Next.js, hosting is Vercel, roster is six games, license is MIT. Build NOT started. Next step on owner go: Phase P0.

---

## §0 — START HERE (agent protocol)

If you are a model picking up this project:

1. Read this section, §2 (scope), §3 (architecture), the current phase in §6, and §4 for whatever game/mechanism you're touching. §4 is the correctness oracle — it is normative.
2. Run `pnpm verify` before starting. Do not build on a red baseline.
3. Build only what the current phase scopes. Any idea outside it goes to the Icebox (§2.4), not into the code.
4. New dependency ⇒ ledger entry (§9) first, with justification. Runtime dependency target is `next` + `react` + `react-dom`, nothing else.
5. At session end: update §6 checkboxes, §9 if decisions were made, §11 session log; bump version in `package.json`; propose a commit message `vX.Y.Z: summary`. Then **stop and wait** — never commit or push without an explicit owner ask. This rule has no exceptions.

### Invariants (the constitution — violating any of these fails review)

- **I1 — Pure engine.** `src/engine/` is framework-free TypeScript: no React, no Next, no DOM, no dependencies, no I/O. All solver arithmetic uses exact rationals (bigint fractions) — no floating point in any correctness path.
- **I2 — Computed, never hardcoded.** Every game-theoretic claim shown to a user (equilibrium, dominance, best response, payoff, classification) comes from engine output. Authored prose may interpret results; it may not assert numbers the engine didn't produce.
- **I3 — Determinism.** All randomness flows through the seeded PRNG in `engine/rng.ts`. `Math.random` and `Date`-derived values are banned outside explicitly whitelisted UI seed-initialization. Any simulation is exactly reproducible from its seed, and share URLs carry full state including seed.
- **I4 — Static and silent.** No backend, no analytics, no cookies, no LLM calls, no third-party requests at runtime. The app is a static export (`output: 'export'`) and makes zero network requests after load. Next.js makes adding a backend easy; this invariant makes it a review failure instead.
- **I5 — Pedagogical honesty.** The reveal never calls non-Nash play a mistake when it best-responds to the actual opponent. Degenerate games are disclosed as degenerate, not silently mishandled. Weak-dominance elimination notes its order-dependence when it applies.
- **I6 — Accessibility is per-phase acceptance, not a final pass.** Keyboard-complete, outcomes announced via ARIA live region, AA contrast, no meaning carried by color alone, full reduced-motion parity (animated reveals have discrete-step equivalents).
- **I7 — Process.** Semver on every commit; ledger entry before any consequential choice; no commit/push without explicit ask.

Mechanized checks for I1/I3/I4 live in `scripts/check-invariants.sh` (grep-based, run in CI from P0): no `from 'react'` or `from 'next'` under `src/engine/`; no `Math.random(` outside `src/engine/rng.ts`; no `fetch(`, `XMLHttpRequest`, `sendBeacon`, or off-origin `<script src=` anywhere in `src/`; and the Next backend-creep guards — no `"use server"`, no `route.ts` handlers or `src/app/api/`, no `middleware.ts`, no imports from `next/headers` or `next/server`.

---

## §1 — Vision & core loop

**One paragraph.** A game-theory sandbox for strategy-curious professionals and students that builds intuition the only way it actually forms: you **act** (make a strategic choice with real stakes on the board), you **observe** (the outcome lands — scores move, the matrix cell lights up, patterns accumulate), and then the theory **reveals** what just happened (why that cell, what you left on the table, what an equilibrium even is — *now* that you've felt it). Payoff matrices in a PDF teach vocabulary; this teaches instinct. The owner (strategy/product professional) is user zero; the product doubles as a career-showcase artifact, so the engine must be provably correct and the surface must have taste.

**The three beats, concretely:**

1. **Act** — a small number of high-stakes buttons ("Hold price" / "Undercut"). No tutorial wall before the first move. The matrix is visible; the theory labels are not yet.
2. **Observe** — the outcome cell flashes (statically highlights under reduced motion), scores tick, the history strip grows, an ARIA live region narrates. Over rounds, patterns become visible *before* they're named.
3. **Reveal** — the Analysis drawer: best-response arrows, dominance walkthrough, Nash badges (with exact fractions for mixed), Pareto frontier, and *your* play measured against both the equilibrium and the best response to what your opponent actually did. The theory arrives as an explanation of your own experience.

**What keeps it out of slop territory:**

- A real solver — exact rational arithmetic, all equilibria of arbitrary user-entered matrices up to 4×4, independently verified and cross-checked against external oracles (§7). Not lookup tables.
- Ruthless roster: six games, each carrying exactly one concept (§4). "One game, one concept" is the editorial razor.
- Pedagogical honesty (I5) — most game-theory toys scold any deviation from Nash; that's wrong and experts notice.
- Restraint in voice and motion: no mascots, no exclamation marks, no confetti. Editorial, print-like visual language.

**Differentiation vs. prior art** (reviewers will know Nicky Case's *The Evolution of Trust*, 2017 — the benchmark in this space): that is a brilliant *linear explorable* for one game with hardcoded outcomes. This is a *sandbox with a live analytical engine*: n×m solving of games the author never anticipated, payoffs editable everywhere with analysis recomputing live, a strategic-structure classifier ("you just turned a dilemma into a trust problem"), and business framing throughout. Complement, not clone — and the README will say so.

---

## §2 — Scope

### 2.1 The five candidate surfaces → three shipped surfaces

The brief's five modes (Play / Simulate / Understand / Build / Learn) collapse into three, because two of them are properties of the system, not destinations:

| Brief mode | v1 disposition |
|---|---|
| Play | **Play** — six games vs. specified opponent personas, session scoring, full reveal. |
| Understand | **Fused into Play** as the Analysis drawer + reveal beats. A separate "Understand" tab would recreate the static-PDF problem we exist to solve. |
| Simulate | **Evolve** — Axelrod-style tournament + replicator dynamics for the iterated PD. The marquee shareable. |
| Build | **Build (lite)** — payoff editing on every game page + a blank custom-game page (2×2 to 4×4), same engine, same reveal, shareable by URL. Nearly free once the engine is generic, and it's the "this is a real analytical tool" proof. |
| Learn | **Embedded, not a mode.** Home page is an ordered arc with a "start here" path; each game opens with a 30-word real-world frame; concepts are explained in place at reveal time, with a glossary. A gated curriculum is v2. |

### 2.2 In v1

- 5 one-shot games (Prisoner's Dilemma, Stag Hunt, Battle of the Sexes, Chicken, Matching Pennies) played in repeated one-shot sessions vs. personas, plus the Iterated Prisoner's Dilemma as a sixth playable (you vs. a strategy, with mystery mode and counterfactual replay).
- Full normal-form analysis engine: pure & mixed Nash (all equilibria, exact fractions), strict/weak dominance with IESDS trace, Pareto frontier, risk vs. payoff dominance (2×2), zero-sum detection, 2×2 strategic-structure classifier, degeneracy detection.
- Tournament (round-robin, 8 classic IPD strategies, seeded) and evolution (discrete replicator dynamics, noise, continuation-probability δ, presets that tell the classic cooperation stories).
- Build-lite with URL-encoded shareable games; seeds in URLs so any simulation result is a reproducible link.
- Methods page: how correctness is established, with references. Portfolio-grade honesty as a feature.
- Static deploy to a public URL, CI-verified, accessible (I6).

### 2.3 Explicitly deferred (v2+), with rationale

- **Extensive-form games** (entry deterrence, ultimatum, backward induction). Requires a second engine and a second visualization idiom (trees); doubles surface area for one more concept. It is the flagship v2 feature — entry deterrence is the single best business-strategy game and deserves a full engine, not a bolt-on. v1 is simultaneous-move normal form only.
- **n-player games** (public goods, commons) — different engine assumptions (symmetric aggregative payoffs), different UI. v2.
- **Learn-as-curriculum** with progression gating; **dark mode** (tokens are structured so it's additive); **PWA/offline**; **finite-population (Moran) dynamics & formal ESS checker**; **spatial evolution grids**; **zero-determinant strategies** (great v2 tease for the Methods page); **human-vs-human** (needs a backend — collides with I4).
- **Any LLM feature.** Probably never: determinism and authored explanation are the brand.
- **Cross-game meta-score / wallet.** The brief's "payoffs accumulate" holds *within* a session; summing utils across different games is game-theoretically meaningless, and experts would wince. Scores are per-session.

### 2.4 Icebox (ideas parked mid-build land here, never in the current phase)

- Robinson–Goforth full 2×2 topology browser; "guess the equilibrium" quiz mode; iframe embed mode for blog posts; per-game OG images; annotation permalinks; ESS checker for 2-strategy contests; dominance-by-mixtures via LP for 3×3+.

---

## §3 — Architecture & stack

### 3.1 Stack decision

**Next.js (current stable, App Router, `output: 'export'`) + React 19 + TypeScript (strict) + Tailwind v4 — fully static, no backend.** Dev/test: Vitest (+ @testing-library/react + jsdom for components, fast-check for properties, coverage), Playwright (+ @axe-core/playwright), ESLint 9 flat + typescript-eslint, Prettier. Package manager pnpm. Node 24 LTS pinned via `.nvmrc`.

**Owner decision 2026-07-13 (D22), superseding the original Vite pick.** The honest trade-off, on the record: Next's server-first machinery goes unused here and its client runtime is heavier than Vite's (~50–80KB extra first-load, budgeted in §3.4); in exchange we get the owner's daily-driver framework (maintenance is a solo affair — fluency wins long-term), file-based routing with clean URLs, the Metadata API for per-route titles/OG (P10 gets simpler), and Vercel preview deployments per PR as a review aid. With static export, nothing about I4, portability, or determinism is compromised. The switch costs zero now (no code exists); it would have cost a rewrite later. The correctness core (`src/engine/`) is framework-agnostic either way (I1).

**Config guardrails (P0):** `output: 'export'`, `trailingSlash: true` (so `out/` serves correctly on *any* static host, not just Vercel), `images: { unoptimized: true }` (no image-optimization server; imagery is inline SVG anyway). Every interactive surface is a client component; server components exist only as thin static shells for metadata.

**Runtime dependencies: `next`, `react`, `react-dom`. That's the list.** No chart library (hand-rolled SVG — accessible, testable via DOM, visually distinctive), no router dep (file-based routing replaces the hand-rolled hash router of the original plan — one less custom module), no state library (context + reducers; the engine is pure functions). Every dep beyond these needs a ledger entry.

### 3.2 No backend — justification

Nothing in v1 needs a server: no accounts, no persistence beyond localStorage (onboarding flag), no multiplayer. Static means free hosting, no ops, no privacy surface, deterministic deploys, and trivially reproducible builds — all four §1 engineering values at once. Shareability is handled by encoding full state (game, payoffs, persona, seed, δ, ε) in URL query strings (versioned codec, §6 P9). `next build` must succeed with static export — any feature that breaks export is out of scope by construction.

### 3.3 Hosting

**Vercel via Git integration** (owner decision 2026-07-13, D23): owner connects the GitHub repo; every push to `main` deploys production, every PR gets a preview URL (deploys happen only on owner pushes, consistent with I7). Clean URLs (`/play/pd`). Repo public on GitHub (the code is part of the portfolio).

**No lock-in clause:** the build output is a plain static `out/` directory; it must remain deployable to GitHub Pages or Cloudflare Pages unmodified (the `trailingSlash` setting exists for exactly this). No Vercel-specific runtime features (functions, middleware, analytics) — enforced by I4's greps.

### 3.4 Module boundaries

```
/                       README.md · CONTEXT.md · CHANGELOG.md · LICENSE
├─ src/
│  ├─ app/              Next App Router — server shells for metadata; interactivity in client components
│  │  ├─ layout.tsx · page.tsx (home arc) · not-found.tsx
│  │  ├─ play/[game]/page.tsx   (generateStaticParams over the six catalog slugs)
│  │  ├─ evolve/ · build/ · methods/
│  ├─ engine/           PURE TS — no React, no Next, no deps, no I/O (invariant I1)
│  │  ├─ rational.ts    exact arithmetic: bigint numerator/denominator, normalized
│  │  ├─ game.ts        NormalFormGame types + helpers (payoffs[row][col] = [uRow, uCol])
│  │  ├─ solve/
│  │  │  ├─ pure.ts         pure NE by brute-force best-response check
│  │  │  ├─ dominance.ts    strict/weak dominance, IESDS with recorded elimination trace
│  │  │  ├─ mixed.ts        support enumeration (≤4×4), degeneracy detection
│  │  │  ├─ twoByTwo.ts     closed-form 2×2 mixed NE (independent cross-check path)
│  │  │  ├─ pareto.ts       Pareto-efficient outcome set
│  │  │  ├─ riskDominance.ts  Harsanyi–Selten product-of-deviation-losses (2×2)
│  │  │  └─ classify.ts     2×2 strategic-structure classifier (§4.8)
│  │  ├─ verify.ts      independent equilibrium verifier — separate code path from solve/
│  │  ├─ repeated/      strategies.ts (FSM specs) · match.ts · tournament.ts · replicator.ts
│  │  ├─ rng.ts         mulberry32 seeded PRNG — the only randomness source (I3)
│  │  └─ catalog/       one file per game: definition + human-authored oracle properties
│  ├─ components/       arena/ · matrix/ · analysis/ · charts/ · ui/   (client components)
│  ├─ state/            per-surface reducers · URL codec
│  ├─ content/          authored copy as typed TS objects: framing, glossary, insight triggers, persona names
│  └─ styles/           tokens.css (custom properties from day 1) · tailwind entry
├─ fixtures/            machine-generated oracle JSON (committed) — see §7.2
├─ scripts/             gen_fixtures.py (dev-only) · check-invariants.sh · bundle-budget.mjs
├─ e2e/                 Playwright specs (run against the exported `out/` on a local static server)
└─ .github/workflows/   ci.yml
```

Key boundary: **`catalog/` colocates each game's definition with its expected properties.** Tests assert `solve(catalog.pd.game) ≡ catalog.pd.oracle`. The oracle is human-authored from theory (§4); machine-generated fixtures (§7.2) independently check the same claims. Two independent sources; the solver must agree with both.

State: URL query strings are the source of truth for anything shareable; localStorage holds only `seenOnboarding`. React state is per-surface reducers; engine calls are memoized pure functions.

Performance envelope (so nobody builds a Web Worker we don't need): worst case is a 4×4 support enumeration (69 support pairs × small exact linear solves — target <16ms, measured not assumed) and an evolution run (8×8 pairwise matches × 20 reps × ~20 rounds to estimate the payoff matrix, then 100 replicator generations of pure arithmetic — well under 100ms). Everything stays synchronous in v1; budget test logs (not gates) timings. Bundle budget: ≤250KB gzipped JS per route (Next runtime overhead acknowledged in D22), warned in CI.

---

## §4 — Game catalog & correctness oracle

Normative section. Every property below is a unit test. Payoff convention: `(row, col)` payoffs; Player 1 = row = "You". All values exact rationals. Every catalog oracle also asserts `degenerate: false` — the detector must not false-positive on canonical games (see P3).

### 4.1 Prisoner's Dilemma — *concept: dominance, and why rational play can be collectively ruinous*

Frame: **price war** (hold price vs. undercut). Actions: Cooperate (C), Defect (D).

|  | C | D |
|---|---|---|
| **C** | 3, 3 | 0, 5 |
| **D** | 5, 0 | 1, 1 |

- D **strictly dominates** C for both players. IESDS trace: eliminate row-C, then col-C → (D,D).
- Unique NE: **(D,D)**, pure. (Dominance-solvable ⇒ no other equilibria, pure or mixed.)
- Pareto-efficient set: {(C,C), (C,D), (D,C)}. (D,D) is strictly Pareto-dominated by (C,C).
- Satisfies 2R > T+S (6 > 5) — mutual cooperation beats alternation in the iterated version.
- Classifier: `dilemma`. Session default: 10 rounds. Personas: Saint (`always:C`), Cynic (`always:D`), Copycat (`tft`), Learner (`fictitious`).
- Key reveal beat: dominance walkthrough → "you were both individually right and collectively poorer — that's the dilemma."

### 4.2 Stag Hunt — *concept: equilibrium selection — trust vs. safety*

Frame: **platform/standards adoption** (bet on the new standard together vs. stick with the legacy). Actions: Stag (S), Hare (H).

|  | S | H |
|---|---|---|
| **S** | 4, 4 | 0, 3 |
| **H** | 3, 0 | 3, 3 |

- No dominance. Two pure NE: **(S,S)** and **(H,H)**.
- Mixed NE: each plays S with probability **3/4** (indifference: 4p = 3). Expected payoff at the mixed NE: 3 each — exactly the safe outcome, a teachable irony.
- Pareto-efficient set: {(S,S)} — it strictly dominates (H,H).
- **(S,S) is payoff-dominant; (H,H) is risk-dominant** (deviation-loss products: (4−3)² = 1 vs. (3−0)² = 9). Surfacing risk dominance is a differentiator — toys never do it.
- Classifier: `coordination (assurance)`. Session: 10 rounds. Personas: Trusting (FSM: start S, mirror your last move), Cautious (FSM: start H; switch to mirroring after you play S twice consecutively), Learner.
- Key reveal beat: "both of you playing it safe *is* an equilibrium. Nothing broke — trust just never formed. Here's what risk dominance means."

### 4.3 Battle of the Sexes — *concept: coordination with conflicting interests — commitment and focal points*

Frame: **post-merger systems integration** — both companies need to land on one platform; each prefers its own. Agreeing on either beats not agreeing at all. Actions: Yours (A — row's favorite), Theirs (B — column's favorite). *(Owner decision 2026-07-13, D24: in roster.)*

|  | A | B |
|---|---|---|
| **A** | 2, 1 | 0, 0 |
| **B** | 0, 0 | 1, 2 |

- No dominance. Two pure NE: **(A,A)** and **(B,B)** — coordination succeeds at both, but the surplus splits differently; the players disagree about *which* equilibrium.
- Mixed NE: row plays A with probability **2/3**; column plays A with probability **1/3** (i.e., each favors their own platform with probability 2/3). Derivation: column indifferent ⇔ p = 2−2p ⇔ p = 2/3; row indifferent ⇔ 2q = 1−q ⇔ q = 1/3.
- At the mixed NE: expected payoff **2/3 each**, and coordination fails with probability **5/9** — "insisting on fairness via randomization" is Pareto-dominated by *both* pure equilibria (1 > 2/3 and 2 > 2/3 for each player). A core reveal beat.
- Pareto-efficient set: {(A,A), (B,B)}.
- Risk dominance: **tie** — Nash products equal ((2−0)·(1−0) = 2 both ways). The engine must report "none/tie"; the reveal explains that theory is silent here, so *commitment and focal points* decide — which is exactly the concept this game carries (distinct from Stag Hunt, where the players agree which equilibrium is better).
- Classifier: `battle`. **Not degenerate** (the off-diagonal payoff ties are between different outcomes, not a degeneracy — regression test against detector false positives, see P3).
- Session: 10 rounds. Personas: Stubborn (`always:B` — never yields), Learner (`fictitious` — consistent commitment trains it to concede; the commitment lesson made playable), Judge (`random:2/3` on own favorite — feel the 5/9 miscoordination of the mixed NE).
- Key reveal beat: vs. the Learner, the player who commits early and consistently wins their preferred equilibrium — first-mover advantage in mixed-motive coordination, no theory lecture needed until the drawer opens.

### 4.4 Chicken (Hawk–Dove) — *concept: anti-coordination, brinkmanship*

Frame: **capacity war** (two firms deciding whether to flood a market that supports one). Actions: Swerve (Sw), Straight (St).

|  | Sw | St |
|---|---|---|
| **Sw** | 0, 0 | −1, 1 |
| **St** | 1, −1 | −10, −10 |

- No dominance. Two pure NE: **(St,Sw)** and **(Sw,St)** — the equilibria are asymmetric; someone must back down.
- Mixed NE: each Swerves with probability **9/10** (indifference: q−1 = 11q−10). Crash probability 1/100. Expected payoff **−1/10** each — worse than mutual swerving, the price of brinkmanship.
- Pareto-efficient set: {(Sw,Sw), (Sw,St), (St,Sw)}.
- Classifier: `anti-coordination`. Session: 10 rounds. Personas: Daredevil (`always:St`), Judge (mixed-NE player, seeded 9/10 Sw), Learner.
- Key reveal beat: the mixed equilibrium computed live, in fractions — and the observation that equilibrium play still burns value.

### 4.5 Matching Pennies — *concept: mixed strategies — why unpredictability is rational*

Frame: **audit vs. evade / penalty kicks**. P1 = Matcher. Actions: Heads (H), Tails (T).

|  | H | T |
|---|---|---|
| **H** | 1, −1 | −1, 1 |
| **T** | −1, 1 | 1, −1 |

- Zero-sum (engine detects and labels). **No pure NE.**
- Unique mixed NE: **(1/2, 1/2) for both**. Game value 0.
- Classifier: `cycle`. Session: 20 rounds (exploitation needs time to show). Personas: Coin (`random:1/2` — the equilibrium player), **Shark** (`markov2` exploiter, §4.7).
- Key reveal beat — the single best intuition moment in the product: after 20 rounds vs. the Shark, "you played Heads 65% of the time and it cost you X. The only mix that can't be exploited is 50/50. *That's* what a mixed equilibrium is." The user feels the theorem before reading it.

### 4.6 Iterated Prisoner's Dilemma — *concept: the shadow of the future*

Stage game = §4.1. Playable: you vs. a strategy for a match (default continuation probability δ = 0.95, expected 20 rounds, seeded so length is reproducible); **mystery mode** hides the opponent's identity until the reveal, which shows its FSM diagram, your cooperation rate, and a **counterfactual replay**: "TFT in your seat would have scored X against this same opponent and seed" (engine rerun — cheap, unique, honest).

**Strategy roster** (all specified as finite-state machines or seeded policies; all deterministic given seed):

| Strategy | Spec |
|---|---|
| Always Cooperate | C forever |
| Always Defect | D forever |
| Tit for Tat | C first, then copy opponent's last move |
| Grim Trigger | C until opponent defects once, then D forever (2-state FSM) |
| Pavlov (win–stay, lose–shift) | C first; repeat your move iff payoff ∈ {T, R}, else switch |
| Generous TFT | TFT, but forgive a defection with probability 1/3 (seeded) |
| Joss | TFT, but sneak a D with probability 1/10 (seeded) |
| Random | C with probability 1/2 (seeded) |

**Exact-transcript tests** (deterministic pairs, asserted move-by-move and on total payoffs): TFT vs TFT → all-C. TFT vs AllD → C then all-D; payoffs (L−1 vs L+4) for length L. Pavlov vs AllD → alternates C,D,C,D…. Grim vs Joss → C-run until Joss's first sneak-D (seed-determined round), then mutual-D tail. Noise ε (per-move flip, seeded) off by default in Play, exercised in Evolve.

### 4.7 Opponent personas (engine policy IDs; display names live in `content/`)

| Policy ID | Spec (must be exactly this — tests depend on it) |
|---|---|
| `always:X` | fixed action X |
| `random:p` | action sampled with probability p (seeded) |
| `tft`, `grim`, `pavlov`, `gtft`, `joss` | as §4.6 |
| `fsm:<name>` | named FSM from catalog (e.g., Cautious, §4.2) |
| `fictitious` | best response to add-1-smoothed empirical mix of ALL your past moves; ties → lowest action index |
| `markov2` | order-2 predictor: contexts = your last 2 moves; add-1 counts; predict your argmax next move (tie → seeded uniform); play the one-shot best response to that point belief; first 2 rounds seeded uniform |

Note: the Battle of the Sexes personas reuse existing policy IDs (`always:B`, `fictitious`, `random:2/3`) — no new engine machinery.

### 4.8 The 2×2 classifier (Build mode's payoff)

Pure function on the ordinal best-response structure of a 2×2 game → one of: `dominance` (dominance-solvable, outcome efficient), `dilemma` (dominance-solvable, outcome strictly Pareto-dominated), `coordination (assurance)` (2 pure NE, players agree which is better), `battle` (2 pure NE, players prefer different NE), `anti-coordination` (2 pure NE off-diagonal), `cycle` (no pure NE), `degenerate` (payoff ties make the class boundary ambiguous). Unit-tested against a hand-built table including all six catalog games and boundary cases. This powers the Build-mode moment: edit PD's temptation payoff from 5 to 2 and the app tells you "this is now an assurance game — two equilibria, trust decides which."

### 4.9 Tournament & evolution (Evolve surface)

- **Match:** IPD, continuation probability δ (slider 0.5–0.995, default 0.95), optional noise ε ∈ [0, 0.1] (per-move flip, default 0), seeded. Match length drawn from the seeded RNG via δ.
- **Tournament:** full round-robin over the §4.6 roster (self-play included), R = 20 repetitions per ordered pair with a deterministic seed schedule derived from a master seed. Output: pairwise mean-payoff matrix (heatmap + accessible table) and ranking. Frozen-seed regression test pins the full result table.
- **Evolution:** infinite-population **discrete replicator dynamics** on strategy shares x: per generation, fitness fᵢ = Σⱼ xⱼ·U(i,j) from the (seeded, precomputed) pairwise payoff matrix; update xᵢ′ = xᵢ·fᵢ / Σₖ xₖ·fₖ. Deterministic given the payoff matrix — the only stochastic step is estimating U. Default 100 generations. No mutation in v1.
- **Presets (each is pedagogy AND a frozen-seed regression test; qualitative assertions verified empirically during the phase, then pinned):**
  1. *Exploitation* — AllC vs AllD → AllD fixates.
  2. *Reciprocity* — add TFT at 10% → cooperation takes over at δ = 0.95.
  3. *Invasion* — 99% AllD + 1% TFT: can a cluster of reciprocators invade? Explore δ threshold.
  4. *Noise* — ε = 5%: TFT erodes (echo feuds); Generous TFT / Pavlov outperform.
  5. *Shadow of the future* — same population, δ = 0.6 vs 0.95: defection vs cooperation. The δ slider is the lever the whole surface exists to teach.

---

## §5 — Interaction, visualization & UX

### 5.1 Surfaces & routes

`/` home · `/play/[game]` (six slugs, prerendered) · `/evolve` (tabs: Tournament, Evolution) · `/build` (custom game) · `/methods` · not-found.

Home is an ordered arc, not a menu: thesis line, then game cards numbered as a suggested path (PD → Stag Hunt → Battle of the Sexes → Chicken → Pennies → IPD → Evolve → Build). No gating — respect adult users.

### 5.2 Game page anatomy (the three beats on screen)

```
┌ Price War · round 4/10 · You 7 — Rival 12 ──────────────────────────┐
│ ARENA                                 │ MATRIX (semantic <table>)   │
│  [ Hold price ]  [ Undercut ]         │   3,3 │ 0,5                 │
│  (buttons; keys 1/2)                  │  ──────┼──────  ← BR arrows,│
│  History: ●●○● you / ○●●● rival       │   5,0 │ 1,1★    NE badge ★  │
│  "You held, rival undercut: 0 vs 5."  │                             │
│  (aria-live narration)                │ [ Analysis ▾ ]              │
└──────────────────────────────────────────────────────────────────────┘
```

- **Act:** choice buttons with the game's verbs (never bare "C/D"); opponent shown as "deciding…" (no artificial delay beyond ~300ms; none under reduced motion).
- **Observe:** outcome cell highlight, score tick, history strip append, live-region sentence. ≤150ms transitions.
- **Reveal:** Analysis drawer, always openable, contents unlocked progressively — after round 1 a "what just happened" line; the full analysis panels from round 3. **Insight moments** (≤2 per game, specified in `content/`) are small toasts pointing into the drawer when a trigger fires — e.g., Pennies: Shark's prediction accuracy > 60% after round 8 → "You're leaking a pattern." Progressive disclosure is the restraint mechanism; the drawer never auto-opens.

### 5.3 Analysis drawer components (shared everywhere, including Build)

1. **Best response** — arrows/underlines per opponent action; sr-only text ("Best response to Hold is Undercut").
2. **Dominance** — IESDS stepper, one elimination per step (discrete steps double as the reduced-motion path); weak-dominance order caveat when applicable (I5).
3. **Equilibria** — NE badges on pure cells; mixed NE as exact fractions and decimals ("Swerve with probability 9/10 (0.90)"). Degenerate games get a plain-language disclosure banner.
4. **Efficiency** — Pareto toggle dims dominated outcomes; the PD's "equilibrium ∉ efficient set" moment.
5. **Your play** — empirical mix vs. NE mix (paired bars); realized payoff vs. (a) best fixed action against opponent's realized sequence — the *hindsight gap*, Δ = maxₐ Σₜ u(a, bₜ) − Σₜ u(aₜ, bₜ) ≥ 0 — and (b) equilibrium payoff. Copy rule (I5): when the opponent didn't play equilibrium, say explicitly that beating the NE payoff is possible and rational.
6. **Structure** (Build + Stag + BoS) — classifier verdict; risk vs. payoff dominance where defined, including the honest "tie — theory is silent, focal points decide" case (BoS).

### 5.4 Evolve visualizations

Stacked-area population chart (SVG): direct band labels at final generation (no color-only legend), hover/focus isolates a band, **generation scrubber** as the primary control (autoplay optional, disabled under reduced motion), story caption per preset, "view as table" fallback (also the screen-reader path). Tournament: pairwise heatmap + ranked bars, same table fallback.

### 5.5 Visual language & accessibility

Editorial/print aesthetic: paper background, near-black ink, one accent; system font stack in P0 (self-hosted Inter as a P10 option — never a fonts CDN, per I4); `font-variant-numeric: tabular-nums` for all payoff numbers. Light theme only in v1 (D19); tokens in CSS custom properties from day 1 so dark mode is additive. Cooperate/defect distinguished by color pair (colorblind-safe blue/orange) **plus** icon shape and label — never color alone.

Accessibility acceptance per phase (I6): keyboard-complete (choices are buttons, matrix is a real `<table>` with headers, drawer is a disclosure with focus management); `aria-live="polite"` round narration; axe scan clean in e2e; AA contrast; `prefers-reduced-motion` honored globally (every animation either disappears or becomes discrete steps — the three-beat loop must work fully with motion off).

### 5.6 Copy voice

Audience: smart professional colleague. Rules: no exclamation marks, no emoji in product copy, no "let's dive in", wry at most once per page. Jargon gets an inline gloss on first use via a `GlossaryTerm` component (~11 terms: Nash equilibrium, dominant strategy, best response, mixed strategy, Pareto efficient, risk dominant, focal point, zero-sum, replicator dynamics, continuation probability, degenerate). Definitions must match Osborne (§12) — one canonical source so definitions never drift. All factual claims in Axelrod/history copy are verified against the primary source during the authoring phase, not recalled from memory.

---

## §6 — Phased build plan

Sizes: S ≈ half a session, M ≈ one, L ≈ one to two (a session = one focused agent run). Total ≈ 14–17 sessions. Every phase ends with the global Definition of Done (§7.5) plus its own acceptance list. Phases are strictly sequential unless noted.

### P0 — Walking skeleton (M)
Scaffold Next.js (App Router, TS strict, `src/` dir, Tailwind v4) with `output: 'export'`, `trailingSlash: true`, `images.unoptimized`; the six routes as static stubs (`/play/[game]` via `generateStaticParams` over the six catalog slugs, `not-found.tsx` for unknown slugs); layout shell + design tokens; version string in footer (read from `package.json`); Vitest (+ RTL/jsdom, coverage, fast-check), Playwright (+ axe) configured to run against the exported `out/` on a local static server (`serve` dev dep, D28); ESLint flat + Prettier; `pnpm verify` (typecheck+lint+unit) and `pnpm verify:full` (+ build + e2e); `scripts/check-invariants.sh` (incl. the Next backend-creep greps, §0) + bundle budget script; `ci.yml`; `.nvmrc`, `.editorconfig`; LICENSE already in repo. Owner connects the repo to Vercel (framework preset Next.js).
**Accept:** `pnpm verify:full` green locally; `next build` exports all routes — **deep-linking directly to `/play/pd/` on the static server works** (the classic exported-app pitfall, caught here); `/play/unknown` serves the 404 page; invariant script wired into CI and failing on a seeded violation (prove it works, then remove the seed); axe clean on shell; after owner pushes + connects Vercel: CI green, live URL serving the shell, PR preview URLs working. Demo: navigate all routes by keyboard.

### P1 — Engine I: exact core (M)
`rational.ts` (bigint fractions: normalize, arithmetic, compare, parse/format); game types; `catalog/` entries for all six §4 games (definitions + oracle objects); `solve/pure.ts`, `dominance.ts` (with elimination trace), `pareto.ts`, `riskDominance.ts`, `classify.ts`; `verify.ts` (independent checker: given a profile, exact best-response verification); `rng.ts`.
**Accept:** every §4.1–§4.5 oracle property is a named passing test (`catalog.oracle ≡ solve(catalog.game)`), including the BoS risk-dominance **tie** case; rational arithmetic property-tested (fast-check: field axioms on random fractions); classifier table test incl. boundary/degenerate cases; engine coverage ≥95% lines, `solve/*` 100%; invariant script clean.

### P2 — First playable: Prisoner's Dilemma (M)
Play route for PD: session loop (10 rounds), personas `always:C`, `always:D`, `tft`, arena + matrix + history + scores, outcome highlight, live-region narration, minimal reveal (dominance + NE badge + session stats from engine output — no hardcoded claims, I2).
**Accept:** e2e: complete a 10-round session keyboard-only; axe clean; reduced-motion emulation shows static-highlight path; unit tests for session reducer; a human can feel beats 1–2–3 (owner demo note).

### P3 — Engine II: mixed equilibria & oracles (L)
`solve/mixed.ts`: support enumeration over equal-size supports (≤4×4), exact linear solves in rationals, non-negativity + off-support deviation checks; **degeneracy detection on structural signals only** — singular support systems, zero-probability/boundary solutions, continuum indicators — **not raw payoff ties** (D27; BoS's off-diagonal (0,0)s must not trip it); `solve/twoByTwo.ts` closed form; fixtures pipeline (§7.2) with `scripts/gen_fixtures.py` + committed JSON.
**Accept:** Pennies (1/2, 1/2), Stag (3/4), Chicken (9/10), and BoS (2/3 · 1/3) mixed NE match §4 exactly as fractions; 2×2 closed form ≡ support enumeration on random 2×2s (fast-check); all committed fixtures match (equilibrium sets equal as exact rationals); a deliberately degenerate fixture is flagged **and all six catalog games assert `degenerate: false`** (false-positive regression); solver timing logged <16ms on 4×4.

### P4 — Reveal layer (L)
The six Analysis-drawer components (§5.3) as shared components; hindsight-gap and mix-vs-NE stat functions in engine (unit-tested formulas); glossary system + terms; honesty rules implemented as copy conditions (I5).
**Accept:** stat functions property-tested (Δ ≥ 0 always; Δ = 0 iff a constant best fixed action was played); drawer fully keyboard-operable, focus-managed, axe clean; IESDS stepper discrete steps = reduced-motion path; PD page now shows the full drawer.

### P5 — Full one-shot roster (L)
Stag Hunt, Battle of the Sexes, Chicken, Matching Pennies pages; personas `fictitious`, `markov2`, `fsm:cautious`, `fsm:trusting`, `random:p`; risk/payoff dominance surfaced on Stag and the tie case on BoS; insight-moment triggers (≤2/game) from `content/`; per-game framing copy (30-word scenarios, fact-checked).
**Accept:** persona unit tests against §4.7 specs (deterministic transcripts with fixed seeds/histories); e2e per game; the Pennies-vs-Shark session demonstrably exploits a biased scripted input in an integration test (prediction accuracy assertion); **BoS commitment test: against a scripted always-A row, the Learner converges to conceding (playing A) within a bounded number of rounds — deterministic**; copy passes voice rules (checklist review).

### P6 — IPD match play (M)
FSM strategy framework + all 8 strategies; match engine (δ, ε, seeds); IPD Play page: match vs. chosen or mystery strategy, FSM diagram reveal, counterfactual replay ("TFT in your seat: X").
**Accept:** exact-transcript tests from §4.6 pass move-by-move and on totals; match engine deterministic given (strategies, seed, δ, ε) — regression-pinned; counterfactual uses the identical seed schedule (test: replaying the actual player's own moves reproduces the actual score exactly); e2e mystery flow.

### P7 — Tournament (M)
Round-robin engine with deterministic seed schedule; Evolve surface, Tournament tab: roster picker, heatmap + ranking + table fallback; Axelrod framing copy (fact-checked against the book).
**Accept:** frozen-seed regression: full pairwise matrix JSON pinned; sanity assertions (AllD vs AllC mean = T exactly; TFT vs TFT mean = R exactly); heatmap has table fallback and axe passes; default-roster winner under default seed documented in the test name.

### P8 — Evolution (L)
Pairwise payoff estimation (seeded), replicator iteration (floats acceptable here **only** because replicator shares are simulation state, not solver claims; document this boundary in code); presets 1–5; stacked-area chart + scrubber + table fallback; δ/ε/share controls; URL state for the whole configuration.
**Accept:** replicator unit tests (2-strategy analytic cases: AllC/AllD shares follow the known monotone path; fixed points at 0/1); preset regression tests under frozen seeds asserting the §4.9 qualitative outcomes with concrete thresholds (e.g., preset 2: TFT+cooperators' combined share >0.9 by generation 100); scrubber keyboard-operable; reduced motion = no autoplay; share URL reproduces a run bit-for-bit.

### P9 — Build mode & share URLs (M)
Payoff editing on every game page (cells become inputs; decimals and fractions accepted; debounced 150ms re-solve); `/build` blank custom game 2×2–4×4 with editable labels; classifier verdict line with change narration ("was a dilemma; now an assurance game"); versioned URL codec on query strings (`?v=1&…`: row-major rational payoffs, labels, persona, seed, δ, ε) with round-trip tests + malformed-input graceful fallback; copy-link share button.
**Accept:** codec round-trip property test (fast-check on random games); malformed/oversized URLs → clean default + notice, never a crash; editing PD's T: 5→2 flips classifier to assurance and adds the second NE badge (integration test straight from §4.8's example); degeneracy banner appears for a genuinely degenerate edit; 4×4 custom game solves within budget.

### P10 — Ship v1.0.0 (M)
Home arc + thesis copy; Methods page (correctness statement, testing approach, references, prior-art credit to Evolution of Trust); first-visit onboarding hint (localStorage); per-route titles + OG via the Next Metadata API + one brand OG image; not-found polish; favicon; full copy edit against §5.6; full a11y audit (axe everywhere + manual screen-reader script through one complete PD session); Lighthouse ≥95 across categories; bundle ≤250KB gz per route; README final — live URL, screenshots/GIF, correctness section, and a **"two-minute tour"** for time-poor reviewers (the three killer moments: Pennies-vs-Shark reveal, PD→assurance payoff edit, the noise preset in Evolve); CHANGELOG; tag v1.0.0.
**Accept:** all of the above as a checklist; owner walkthrough sign-off; `pnpm verify:full` green in CI on the tagged commit; `out/` also deploys unmodified to a non-Vercel static host (portability spot-check, §3.3).

---

## §7 — Test & verification spine

### 7.1 Layers

1. **Human-authored oracle** — §4 properties, encoded in `catalog/` and asserted against solver output. Exact rational equality, not approximate.
2. **Independent verifier** — `verify.ts` re-checks any claimed equilibrium via best-response conditions, in a code path disjoint from the solver. Property test (fast-check): for random matrices (2×2–4×4, integer payoffs), everything the solver returns passes the verifier (**soundness**), and pure-NE output matches brute force.
3. **External cross-oracle** — `scripts/gen_fixtures.py` generates equilibrium sets for a fixture battery (the six catalog games + ~30 random + known degenerate cases) using **Gambit** (`gambit-enummixed`, rational output preferred) with **nashpy** fallback (floats → nearest small rational → certified exactly by our verifier; completeness checked by count agreement). Fixtures committed as JSON with the generator, so **completeness** is checked against an independent implementation. The epistemics (what certifies what) are documented in the fixture README and on the Methods page.
4. **Closed-form cross-check** — 2×2 mixed formula vs. support enumeration on random 2×2s.
5. **Simulation determinism** — exact-transcript tests for deterministic strategy pairs; frozen-seed regression for anything stochastic (tournament table, evolution presets); qualitative preset assertions with concrete thresholds.
6. **UI** — component tests for reducers/codec; Playwright e2e per surface (happy path, keyboard-only, reduced-motion, axe) against the exported static build.

### 7.2 Known limitation, stated honestly

Support enumeration over equal-size supports finds **all** equilibria of nondegenerate bimatrix games. Degenerate games (which users will create — round numbers produce ties) can have equilibrium components; v1 detects degeneracy structurally (D27), reports pure NE + sample mixed NE, and says so in the UI (I5) and on the Methods page. Pure-strategy dominance only in v1 (dominance by mixed strategies is complete for 2×2 automatically; for 3×3+ it's an LP — Icebox), with the boundary documented.

### 7.3 CI

GitHub Actions, Node 24, single job: install → check-invariants → typecheck → lint → unit (+coverage gate on engine) → `next build` (static export) → bundle budget (warn) → Playwright (chromium, against `out/`). Deploys are Vercel's Git integration (production on `main`, previews on PRs) — no deploy workflow of our own. CI must be green before any phase is offered for sign-off.

### 7.4 Phase sign-off protocol

(a) CI green; (b) phase acceptance checklist all ✓ in §6; (c) owner eyeballs the demo notes — the Vercel PR preview URL is the review surface; (d) version bump + CHANGELOG entry; (e) owner explicitly says commit/push. Reviewer (Opus) protocol: run `pnpm verify:full` fresh; walk the acceptance list; run the invariant greps; spot-check that new UI claims trace to engine output (I2); check §9/§11 were updated.

### 7.5 Global Definition of Done (every phase)

Types strict (no `any` without a comment); tests for all new engine/stat logic; a11y criteria of §5.5 met for new UI; reduced-motion path exists; docs updated (this file: §6 checkboxes, §9 if decisions, §11 log); version bumped; CHANGELOG line written; no new deps without ledger entry; invariant script clean.

---

## §8 — Repo conventions

- **README.md** (public): what/why, live URL, thesis, feature tour, correctness statement, stack, quickstart, license. Audience: recruiters, peers, users.
- **CONTEXT.md** (this file): agent protocol, plan, oracle, ledger, backlog, session log. Audience: the owner and build/review models. The two never merge (owner's standing workflow).
- **CHANGELOG.md**: one line per version, kept from the first code commit (P0).
- **Versioning:** semver on every commit, pre-1.0 scheme: **minor** = phase completion, **patch** = everything else; `v1.0.0` at P10. Version lives in `package.json` (from P0) and the UI footer. Commit messages: `vX.Y.Z: imperative summary`.
- **Branching:** trunk-based, `main` only; work sessions produce a reviewed working tree; the owner triggers every commit/push (I7). Short-lived branches only if two agents ever work in parallel (not planned for v1).
- **Dependency policy:** runtime = next, react, react-dom; dev deps allowed with ledger entry; anything touching I4 (network, telemetry) is rejected outright.

---

## §9 — Decisions ledger

Append-only. When a decision is overturned, the old row stays and points to its successor — the ledger is the project's memory, including of its reversals. This section absorbs the retired "veto" section (D26) and grows as future decisions land.

| # | Decision | Alternatives considered | Why |
|---|---|---|---|
| D1 | ~~Vite + React SPA~~ → superseded by D22 | Next.js; vanilla ESM; SvelteKit | Original lean-toolchain pick; overruled by owner pre-build (zero switch cost at that point) |
| D2 | Fully static, no backend | Serverless for share-state | Nothing needs a server; free, private, deterministic (I4) |
| D3 | ~~GitHub Pages + hash routing~~ → superseded by D23 | Cloudflare Pages; Vercel | Original zero-extra-vendor pick; overruled by owner for Vercel + clean URLs |
| D4 | Exact bigint-rational arithmetic in solver | Floats + epsilon | Correctness is the product; exact equality in tests; fraction display is a feature |
| D5 | Support enumeration, ≤4×4 cap, degeneracy disclosed | Lemke–Howson; LP/LCP solvers | Exhaustive & exact at this size; L–H finds one equilibrium, we want all; cap keeps it honest (§7.2) |
| D6 | v1 = normal form only; extensive form is v2 flagship | Both engines in v1 | Second engine + tree UI doubles scope; entry deterrence deserves a real build |
| D7 | ~~BoS cut from roster~~ → superseded by D24 | Keep BoS | Cut on claimed overlap with Stag/Chicken; owner overruled — and the distinct concept slot (commitment/focal points, risk-dominance tie) is the better analysis |
| D8 | Personas as specced policies (FSM/fictitious/markov2), ≤1 knob each | Configurable AI parameter panels | Testability + restraint; config-soup is slop |
| D9 | IPD engine = seeded simulation | Markov-chain exact expected payoffs | Matches Axelrod's actual method; simpler for build models; transcripts still exactly tested; exact chains = v2 |
| D10 | Evolution = infinite-population discrete replicator, no mutation | Moran process; mutation | Deterministic given payoff matrix; the classic stories need nothing more (§4.9) |
| D11 | Hand-rolled SVG viz; no chart/state deps | Recharts/d3; zustand | a11y via real DOM; distinctive look. (Router clause superseded by D22 — Next file routing; no-chart/no-state-lib stands) |
| D12 | Seeded PRNG (mulberry32) everywhere | Math.random | Reproducibility, share URLs, frozen-seed tests (I3) |
| D13 | Catalog colocates game + human oracle; fixtures machine-generated | Tests scattered per suite | Single source of truth; two independent oracles must agree (§3.4, §7.1) |
| D14 | pnpm; Node 24 LTS pinned | npm | Strict node_modules keeps agents honest about deps; trivial to swap |
| D15 | fast-check property tests (dev dep) | Example-based only | Solver soundness is a property, not an example |
| D16 | No LLM features, no analytics, ever in v1 | Privacy-friendly counter | Determinism is the brand; zero third-party requests is a verifiable claim (I4) |
| D17 | Copy as typed TS objects in `content/` | MDX | No extra toolchain; content is testable (glossary presence, trigger conditions) |
| D18 | System fonts in P0; self-hosted Inter as P10 option | Google Fonts CDN | I4; tabular-nums available either way |
| D19 | Light-only editorial theme in v1; tokens ready for dark | Both themes now | Every viz ×2 review cost; deliberate print identity |
| D20 | Per-session scores only; no cross-game wallet | Brief's "payoffs accumulate" read globally | Summing utils across games is theoretically meaningless |
| D21 | Repo public from day 1 | Private until polished | The plan and engine are the portfolio; public history shows discipline |
| D22 | **Next.js App Router + `output: 'export'`** (owner, 2026-07-13) | Vite (D1) | Owner fluency/preference; zero cost pre-code; static export preserves I4 + portability; file routing, Metadata API, PR previews are real wins; ~50–80KB runtime overhead accepted and budgeted (§3.4) |
| D23 | **Vercel hosting via Git integration** (owner, 2026-07-13) | GitHub Pages (D3); Cloudflare Pages | Clean URLs, PR preview deploys; no-lock-in clause: `out/` stays portable to any static host (§3.3) |
| D24 | **Battle of the Sexes in v1 roster — six games** (owner, 2026-07-13) | Five-game roster (D7) | Concept slot is genuinely distinct: coordination with conflicting interests, commitment, focal points, risk-dominance tie (§4.3) |
| D25 | **MIT license, © Sidakpreet Singh** (owner, 2026-07-13) | Other licenses | Portfolio reach; LICENSE at repo root from v0.0.1 |
| D26 | Veto section retired; owner decisions recorded here from now on (owner, 2026-07-13) | Keep separate veto list | One growing ledger is the living record; pre-build veto round served its purpose |
| D27 | Degeneracy detection on structural signals only (singular systems, boundary/zero-prob solutions, continuum indicators), never raw payoff ties | Conservative tie-based flagging | Tie-based flagging false-positives on canonical BoS; all six catalog oracles assert `degenerate: false` as a regression (P3) |
| D28 | `serve` dev dep to run e2e against the exported `out/`; bundle budget 250KB gz/route | `next start` (doesn't serve exports); no budget | Exported output is what ships; budget acknowledges Next runtime (D22) honestly |

---

## §10 — Open questions & owner actions

1. **Create the GitHub repo** (public, D21) and add the remote; **connect it to Vercel** (framework preset Next.js). Repo name: `gt-sandbox` is fine until naming lands.
2. **Product name.** README currently titled "Game Theory Sandbox" — serviceable, and it can ship under that. Shortlist if you want a name with more edge (my lean first): **Best Response**; Equilibrium Lab; The Payoff; Common Knowledge; Tit for Tat. **Deadline: end of P9** — P10 bakes the name into metadata, OG image, and README.
3. **Accent color preference** welcome before P0; otherwise I spec a restrained, colorblind-safe blue/orange pair in the P0 tokens.
4. **Give the P0 go** when ready — plan is owner-approved as of 2026-07-13; nothing else blocks the first build session.
5. Standing reminder: every commit/push needs your explicit ask, including phase completions.

Resolved 2026-07-13: stack (D22), hosting (D23), roster (D24), license (D25), veto process retired (D26).

---

## §11 — Session log (append-only)

- **2026-07-12 — S0 (planning).** Commissioned v1 plan. Wrote CONTEXT.md (this document), README.md, .gitignore; initialized repo. Decisions D1–D21 opened; pre-build veto list issued for owner review.
- **2026-07-13 — S1 (owner review & revision).** Owner veto pass: Next.js over Vite (D22), Vercel over GitHub Pages (D23), Battle of the Sexes into the roster (D24), MIT license © Sidakpreet Singh (D25); veto section retired into the ledger (D26). Plan revised throughout for the Next.js static-export architecture and the six-game catalog (BoS oracle added, §4.3); degeneracy detection refined against false positives (D27); Next backend-creep invariant greps, P0 deep-link/prerender acceptance, P10 "two-minute tour", and a P9 naming deadline added during review. LICENSE file added. Repo re-initialized at owner request; commit `v0.0.1: Init + Readme + Context` re-issued. Next: owner P0 go (§10.4).

---

## §12 — References (for authored content & definitions)

- Osborne, *An Introduction to Game Theory* — canonical definitions (glossary source of truth).
- Axelrod, *The Evolution of Cooperation* (1984) — tournament framing; all copy claims fact-checked against it.
- Nowak, "Five Rules for the Evolution of Cooperation" (2006).
- Harsanyi & Selten (1988) — risk dominance.
- Robinson & Goforth, *The Topology of the 2×2 Games* (2005) — classifier grounding.
- Sigmund, *Evolutionary Games and Population Dynamics* — replicator formalism.
- Gambit & nashpy — external oracle tooling (§7.1).
- Nicky Case, *The Evolution of Trust* (2017) — prior art, credited on the Methods page.
