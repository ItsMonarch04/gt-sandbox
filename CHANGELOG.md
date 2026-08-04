# Changelog

## v0.14.10 — 2026-08-05

- Add pure classroom aggregation helpers and the assignment-link builder that reuses the existing game URL codec.

## v0.14.9 — 2026-08-03

- Add the auction play surface with format-specific reveals and prerendered `/auctions/[format]` routes.

## v0.14.8 — 2026-08-03

- Add the auction session reducer with persona selection and authored content for the three auction formats.

## v0.14.7 — 2026-08-03

- Add exact common-value winner's-curse analysis and the auction engine unit suite.

## v0.14.6 — 2026-08-03

- Add the private-value auction engine core and an exact second-price weak-dominance demonstration with strictly better witnesses.

## v0.14.5 — 2026-08-02

- Add the `/repeat` surface with folk-theorem region visualization, strategy controls, authored content, and component tests.

## v0.14.4 — 2026-08-02

- Generalize iterated play beyond IPD via a Cooperate/Defect symbol space with per-game action maps and mean-payoff Pavlov aspiration.

## v0.14.3 — 2026-08-02

- Add an exact folk-theorem engine for 2×2 games: feasible payoff hull, mixed-strategy minimax values, and grim-trigger discount thresholds.

## v0.14.2 — 2026-08-02

- Add the hot-seat arena UI with handover concealment, keyboard flow, analysis perspective, and prerendered `/hot-seat/[game]` routes.

## v0.14.1 — 2026-08-02

- Add the hot-seat session reducer (commit-and-conceal phases) and authored content for the five 2×2 hot-seat games.

## v0.14.0 — 2026-08-02

- Add the versioned `gt-sandbox/session` export schema and unit tests shared by hot-seat, iterated, auction, and classroom surfaces.

## v0.13.5 — 2026-07-26

- State sync: bump the Portfolio Version marker to `v0.13.5` across `package.json`, README, and this changelog, and refresh the `CONTEXT.md` handoff to the pushed, in-sync `origin/main` state. No functional change.

## v0.13.4 — 2026-07-26

- Record the TRIAGE.md handoff: strike Learner from §4.1's PD persona list, add D50–D54 to the ledger, log S25 in §11, synchronize the version surfaces, and remove `TRIAGE.md` now that every finding it enumerates lives in the tree or the ledger.

## v0.13.3 — 2026-07-26

- Tighten evolution URL decode bounds to `repetitions` 1–20 and `roundCap` 1–1000 with a rejection test, and match evolution presets field-wise so decoded shared URLs resolve back to their preset name.

## v0.13.2 — 2026-07-26

- Branch the Nash-equilibrium panel on the already-computed degeneracy witness so degenerate games get an equilibria-family disclosure instead of a completeness claim (with a new component test), memoize the drawer's relabeled game and mixed analysis, pluralize the workbench verdict, and document the pareto argument-order.

## v0.13.1 — 2026-07-26

- Hydrate Play-route sessions from the shared URL on mount: each arena decodes persona (and, for IPD, δ and ε) plus seed and dispatches a validated `hydrate` action, with a Playwright test covering PD persona restore.

## v0.13.0 — 2026-07-26

- Guard the invariant checker against a missing ripgrep and unify the Math.random scan under `src/`; fix the narrow-viewport Analysis drawer and Build workbench reflow at 320 px and 200 % zoom; add a client-side route error boundary.

## v0.12.4 — 2026-07-23

- Record the final verification outcome and clarify the release-only browser-run status.

## v0.12.3 — 2026-07-23

- Format the launch browser specification and complete the verified audit handoff.

## v0.12.2 — 2026-07-23

- Add a raster social card and finalize audit/release handoff documentation.

## v0.12.1 — 2026-07-23

- Correct static-route accounting in the bundle audit and remove the obsolete Play fallback.

## v0.12.0 — 2026-07-23

- Harden share-link bounds and simulation metadata validation, and prevent empty validation alerts.

## v0.11.3 — 2026-07-23

- Add release documentation, route-wide accessibility scans, and the P10 handoff.

## v0.11.2 — 2026-07-23

- Add responsive and forced-colors safeguards with launch-specific browser acceptance coverage.

## v0.11.1 — 2026-07-23

- Add the Methods route, route-level metadata, and first-party icon and social-card assets.

## v0.11.0 — 2026-07-23

- Add the launch home arc, first-visit onboarding preference, and foundation launch styling.

## v0.10.4 — 2026-07-22

- Add Build direct-link and 4×4 reflow browser coverage, and complete the P9 handoff.

## v0.10.3 — 2026-07-22

- Embed the exact editable game surface on every Play route with each route’s reproducibility extras.

## v0.10.2 — 2026-07-22

- Add the Build surface, reusable full Analysis drawer, and responsive payoff-matrix presentation.

## v0.10.1 — 2026-07-22

- Add the bounded editable Game Workbench with exact live analysis, validation, copying, and focused component coverage.

## v0.10.0 — 2026-07-22

- Add the bounded versioned custom-game URL codec, exact rational parsing limits, and round-trip property coverage.

## v0.9.4 — 2026-07-22

- Add exported-browser Evolution coverage and complete the P8 handoff.

## v0.9.3 — 2026-07-22

- Add the accessible Evolution tab, population chart, controls, table fallback, and component coverage.

## v0.9.2 — 2026-07-22

- Add bounded evolution URL state, malformed-link fallback, and exact run reproduction coverage.

## v0.9.1 — 2026-07-22

- Add the reviewed evolution calibration corpus, preset content, and frozen-result regression coverage.

## v0.9.0 — 2026-07-22

- Add the seeded IPD replicator engine, analytic dynamics coverage, and guarded zero-fitness handling.

## v0.8.2 — 2026-07-21

- Add exported-browser Tournament acceptance coverage and complete the P7 handoff.

## v0.8.1 — 2026-07-21

- Add the interactive Tournament surface, roster picker, ranking, heatmap, exact-table fallback, and component coverage.

## v0.8.0 — 2026-07-21

- Add the seeded IPD tournament engine, exact pairwise fixture, and frozen regression coverage.

## v0.7.3 — 2026-07-20

- Add exported-browser mystery-flow coverage and complete the P5/P6 handoff.

## v0.7.2 — 2026-07-20

- Add the IPD play arena, mystery-strategy reveal, and same-environment TFT counterfactual.

## v0.7.1 — 2026-07-20

- Add the seeded IPD session reducer, opponent selection content, and match-flow coverage.

## v0.7.0 — 2026-07-20

- Add event-addressed IPD randomness, eight strategy specifications, and exact match-engine transcripts.

## v0.6.2 — 2026-07-20

- Add exported-browser coverage for the full P5 one-shot roster.

## v0.6.1 — 2026-07-20

- Add the four one-shot game arenas, live analysis, and zero-sum disclosure.

## v0.6.0 — 2026-07-20

- Add deterministic one-shot personas, shared session state, and the P5 core test suite.

## v0.5.3 — 2026-07-19

- Add exported-browser drawer acceptance coverage and complete the P4 handoff.

## v0.5.2 — 2026-07-19

- Integrate the full progressive reveal with the Price War matrix and add focused interaction coverage.

## v0.5.1 — 2026-07-19

- Add the reusable progressive Analysis drawer, glossary primitives, and supporting reveal styles.

## v0.5.0 — 2026-07-19

- Add exact session statistics for empirical mixes and signed fixed-action hindsight, with property and Matching Pennies regressions.

## v0.4.3 — 2026-07-19

- Commit the Gambit fixture corpus and manifest, enforce its byte-integrity check, and complete P3 acceptance.

## v0.4.2 — 2026-07-19

- Add the isolated, reproducible Gambit fixture generator and its local image build definition.

## v0.4.1 — 2026-07-19

- Add exact mixed-equilibrium catalog oracles, degeneracy regression coverage, and randomized 2×2 cross-checks.

## v0.4.0 — 2026-07-19

- Add exact mixed-equilibrium support enumeration, a closed-form 2×2 cross-check, and independent mixed verification.

## v0.3.2 — 2026-07-18

- Add exported-browser acceptance coverage and complete the P2 handoff.

## v0.3.1 — 2026-07-18

- Add the interactive Price War arena, engine-derived reveal, and component coverage.

## v0.3.0 — 2026-07-18

- Add deterministic Prisoner's Dilemma policies, session state, and reducer coverage.

## v0.2.3 — 2026-07-18

- Normalize final P1 formatting and refresh the committed handoff.

## v0.2.2 — 2026-07-18

- Add strategic-analysis solvers, deterministic RNG, enforced engine coverage, and the P1 handoff.

## v0.2.1 — 2026-07-18

- Add the six-game catalog, pure equilibrium solver, Pareto analysis, classifier, and verifier.

## v0.2.0 — 2026-07-18

- Add exact rational arithmetic, normal-form game construction, and focused engine tests.

## v0.1.3 — 2026-07-18

- Add static-host security, exported-site verification, CI, and the completed P0 handoff.

## v0.1.2 — 2026-07-18

- Add the Vitest setup, coverage command, and shell unit test.

## v0.1.1 — 2026-07-18

- Add static routes, generated game paths, and route-shell styling support.

## v0.1.0 — 2026-07-18

- Add the Next.js static-export foundation and root shell.
