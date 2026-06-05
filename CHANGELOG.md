# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

Pending the next published release.

### Fixed — `parametersSchema` rejected by the host validator

- `tools/build-custom-tools.mjs` previously shipped `parametersSchema: {}` for the static reference tool baked into every ruleset bundle. Current Marinara host builds reject that with:
  `[tools] Skipping custom tool "<id>_reference" with invalid parameter schema: parametersSchema must define root "type": "object" or include object "properties" {}`
  The shipped tool was silently dropped on install, so the reference tool never reached the model.
- The generator now emits `parametersSchema: { type: "object", properties: {} }` — semantically "no parameters" with the explicit JSON Schema shape the validator expects. Every bundle in `rulesets/*/bundle.json` has been rebuilt to pick up the fix.

## [0.3.0] - 2026-05-22

Canonical agent pool documented as the recommended path (the legacy five remain available — RP-mode users CAN toggle in Settings → Agents), three new per-system parallel-phase overlays land, two new reference rulesets (Blades in the Dark, Genesys), and the Vector 8 per-chat scenario-default install path closes. Non-breaking — companion to GM-mode v0.5.0 which took the more aggressive path (deleted legacy four, all `enabled:true`) because GM-mode has no per-agent toggle UI.

### Added — per-system parallel-phase overlays (2026-05-22)

- `rulesets/exalted3e/agents/anima-banner-monitor.md` — `parallel`-phase `context_injection` agent tracking each Solar/Lunar/Sidereal/Dragon-Blooded character's anima banner level based on Peripheral mote spend across the scene. Runs alongside the narrator without blocking — zero added per-turn latency.
- `rulesets/exalted3e/agents/charm-cooldown-tracker.md` — `parallel`-phase agent scanning recent turns for Charm activations and emitting a per-scene cooldown summary.
- `rulesets/vtmv20/agents/blood-pool-tracker.md` — `parallel`-phase agent tracking each Kindred character's current Blood Pool, per-turn generation cap, and recent significant changes.
- Per-system parallel overlays cannot be folded into universal agents because the resources they track exist only in their system. They are emitted alongside the universal pool in `agents.json`.

### Added — Phase honoring in the build pipeline (Vector 7 enabler, 2026-05-22)

- `tools/build-agents.mjs` and `tools/build-bundle.mjs` both gain `extractPhase()` which honors a sub-agent's `**Phase:**` declaration verbatim (`pre_generation` | `post_generation` | `parallel`), falling back to `pre_generation` only if no declaration is found. Earlier versions hardcoded `pre_generation`, which silently downgraded the new `parallel`-phase agents and broke the V7 path.

### Added — two new reference rulesets (since v0.2.2)

- **Blades in the Dark** (`blades-in-the-dark`) — narrative-handled resolution mode (Forged in the Dark mechanics).
- **Genesys** (`genesys`) — narrative-handled resolution mode (Star Wars / Genesys narrative dice).
- v0.3.0 ships twelve total reference rulesets: blades-in-the-dark, coc7e, dnd5e, exalted3e, fate-core, genesys, gurps-lite, lasers-and-feelings, pathfinder2e, stewpot, trophy-dark, vtmv20.

### Added — Vector 8 per-chat scenario-default install (2026-05-22)

- `tools/build-scenario-default.mjs` already shipped `bundle.scenarioDefault` (introduced in 0.2.2). The Vector 8 close-the-loop install step now lives in `extension/RPG-Extension-RP-Mode.js`: after `installBundle()` resolves, the helper prompts the user once via `window.confirm` and PATCHes `chatMeta.groupScenarioText` on the active chat if approved. The install pipeline never throws from this step.
- `coc7e`, `dnd5e`, `pathfinder2e`, `vtmv20` ruleset.json files gained `scenarioDefault` blocks alongside the existing `exalted3e` derivation.

### Docs — canonical-vs-legacy paths documented across the build docs (2026-05-22)

- `README.md`: added **"Agent layout (canonical pool + RP-mode toggle policy)"** section that documents the new layout and explicitly contrasts RP-mode's `enabled:false` + Settings → Agents toggle against GM-mode's `enabled:true`. The sub-agent reference table was rewritten to show canonical + legacy + per-system parallel overlays separately with "pick one path" guidance. File-tree comment in the repo-layout section updated.
- `AUTHORING-PROMPT.md`: optional-sub-agents section rewritten with both paths + parallel overlays + GM-mode alignment-break note.
- `docs/BUILDING.md`: `build-agents.mjs` purpose updated for both paths; **"Agent consolidation"** section replaced with **"Agent paths (RP-mode toggleable model)"** — three tables (canonical / legacy / parallel) + v0.4.x migration steps + GM-mode alignment-break note.
- `docs/ENGINE-CONSTRAINTS.md`: optional sub-agents section rewritten with both paths + parallel overlays + "pick one" warning.
- `docs/AGENT-COMPATIBILITY.md`: file header updated; **"Our 5 sub-agents (recap)"** replaced with three sub-tables (canonical / legacy / parallel) with phase columns.
- `docs/AUTHORING-PHASE-6.md`: `combat-adjudicator`'s round-ended-signal reference generalized to "active combat agent (combat-overseer on canonical, combat-adjudicator on legacy)".
- `package.json` description rewritten to describe the two-path agent architecture + parallel overlays.
- `schema/bundle.schema.json` `additionalAgents.description` rewritten with both paths and parallel overlays.
- `schema/ruleset.schema.json`: `roundCounters` + `resetOn` descriptions generalized to "active combat agent"; `trackPenaltyKind` description generalized to "active state agent (context-fuser on canonical, state-reminder on legacy)".

### Migration — non-breaking upgrade path

- v0.2.x users can continue to run the legacy five sub-agents (`combat-adjudicator`, `npc-bookkeeper`, `lore-query`, `state-reminder`, plus `state-mutator`) through Marinara Settings → Agents. The build pipeline still emits all of them into `agents.json`.
- To opt into the canonical path: import the new `agents.json` via Marinara's Import Agents dialog, disable the four legacy pre-gen agents, enable `combat-overseer` + `context-fuser` (keep `state-mutator` enabled). Per-turn cost drops from 4 pre-gen + 1 post-proc → 2 pre-gen + 1 post-proc (~40% reduction).
- Per-system parallel overlays opt-in independently — Exalted players can enable `anima-banner-monitor` + `charm-cooldown-tracker`; VTM players can enable `blood-pool-tracker`.
- Release packaged at `releases/v0.3.0/` + matching `releases/Marinara-RPG-RP-Mode-Extension-v0.3.0.zip` (791K).

## [0.2.2] - 2026-05-14

Trinity grouping, Merits & Flaws section, OpenD6 dogfood remediation, parity fixes, and documentation completeness pass.

### Added — V20 trinity skill grouping + Merits & Flaws section (2026-05-14)

- Framework JS in `extension/RPG-Extension-RP-Mode.js` now consumes `state.ruleset.abilities.groups[]` and buckets a ruleset's `skills[]` under named subheaders. V20 ships with the canonical Talents / Skills / Knowledges trinity (10 + 10 + 10) sourced from the official 4-Page Neonate sheet. Skills not listed in any group fall back under an "Other" subheader. Mechanism generalizes to W20, M20, and any future system that wants D&D-by-attribute-style grouping (config-only — no framework change).
- New `meritsFlaws` section type rendered between `backgrounds` and `inventory` when a ruleset declares it in `sections.order`. V20 ships with it enabled. Built-in renderer with two subgroups (Merits / Flaws), per-row name + type (Physical / Mental / Social / Supernatural) + points (1-7) + remove button. Storage at `state.sheet.meritsFlaws[]` with explicit `mergeSheet` persistence whitelist. Section name is generic — any system can reuse.
- Schema enums extended in both `sections.order` and legacy `sheetSections` to include `"meritsFlaws"`.

### Fixed — cross-repo parity drift (2026-05-14)

- **P2 lorebook label fix**: `rulesets/fate-core/lorebook.json` and `rulesets/vtmv20/lorebook.json` referenced the GM-mode agent label as `MRRP:` when it should be `MRR:`. Cross-mode references corrected; vtmv20 also clarifies that GM-mode emits `[mrr-state: ...]` tags while RP-mode emits `[mrrp-state: ...]`. Affected bundles rebuilt.

### Updated — authoring docs to match current 9-mode schema (2026-05-14)

- `docs/AUTHORING-PHASE-6.md` renamed scope to "Phase 5 / 6 / 7 schema additions"; mode list extended from 7 to 9 (added `dice-pool-sum` with Wild Die and `narrative-handled`); new sections cover `pipGranularity` (OpenD6 sub-die precision), `effects.onSpend` (spend-driven bonuses), `roundCounters[]` (per-round combat penalties), `derivedStats[].track[].penaltyKind` (flat / dice), `abilities.groups[]` (trinity), and `meritsFlaws` (V20-anchored section). New "⚠️ Before you start" callout instructs authors to ask Kenhito in the Marinara Extension community thread if their dice mechanic isn't one of the nine supported modes.
- `AUTHORING-PROMPT.md` updated to advertise all 9 modes + new schema features; "⚠️ Before authoring" callout added.
- `docs/AUTHORING.md` and `docs/ADDING-RULESETS.md` updated with 9-mode tables and the "ask Kenhito" escalation path.
- `docs/ENGINE-CONSTRAINTS.md` updated with 9-mode coverage.

### Updated — README with system-grouped dice list + character-save volatility section (2026-05-14)

- `README.md` now lists the 9 supported resolution mechanics grouped by tabletop family (d20 / Storyteller d10 / Percentile / PbtA / Fate / Roll-under / Stance-modal / OpenD6 / Narrative-handled), with each family naming a sample of supported systems.
- New `## Character data persistence` section explains how the save / load / rename / + / x controls work, embeds a screenshot of the controls (`docs/screenshots/character-save-controls.png`), and emphasizes that localStorage is volatile — players should hit `save` regularly to download a JSON file as the master copy. Lists 7 scenarios where localStorage can be wiped.

### Added — Trophy Dark + Stewpot rulesets (stance-modal-pool generalization) (2026-05-10)

Two additional rulesets that exercise `stance-modal-pool` beyond L&F — proves the mode generalizes to the polarized-pool family.

- **NEW `rulesets/stewpot/ruleset.json`** (148 lines, both repos byte-identical): Tim Hutchings-tradition slow-life village RPG. **Clean fit for stance-modal-pool** — same shape as L&F with different theme. Single stat `Capability` (2-5), stance `stillness/under` (observing, recovering) vs `action/over` (making things happen), `poolFormula: 2 + (invested ? 1 : 0) + helping_neighbors`, `exactMatch: PERFECT DAY` (countsAsSuccess: true), 4 outcome tiers (`stalled/small-grace/good-day/harvest`). 5 skills (Garden, Cook, Mend, Listen, Persuade). 1 derived stat (`Hearth`, max 5 bar, refills on Perfect Days).
- **NEW `rulesets/trophy-dark/ruleset.json`** (176 lines, both repos byte-identical): Jesse Ross / 24XX-family horror RPG (treasure hunters in a haunted wood). **Approximated fit** — Trophy Dark's RAW is a dual-pool comparison mechanic (light pool + dark pool rolled in parallel, dark face > highest light inflicts Ruin) which `stance-modal-pool` cannot fully express. The forced fit uses `stat: "Risk"` held fixed at min=max=3 (the 24XX-family Risk-line invariant), `stance: light/over` (faces > 3 = success) + `stance: dark/under` (approximation; GM rolls dark dice as a separate roll under the same Risk and narrates "dark overrules light" out-of-band per gm-agent guidance). 3 outcome tiers (`devils-bargain/at-a-cost/clean`). `exactMatch.name: "EDGE OF RUIN"` (countsAsSuccess: false — narrative flag only). Derived stats: `Ruin` (max 6 bar — the corruption track) + `Burdens` (accepted-devil's-bargain tracker, narrative). 3-4 sample Pursuits + Backgrounds.
- **Trophy Dark deviations honestly documented inline** in `summary`, both stance `description` fields, all `outcomeTiers[].narration` fields. **A faithful Trophy Dark mode would need a new `dual-pool-comparison` resolution branch** with `lightStance / darkStance / overruleRule` fields — flagged for a future schema cycle. Trophy Dark also proves stance-modal-pool needs to accept `min === max` as a valid degenerate (verified PASS; a "fixed stat" hint field would be a future ergonomic).
- **Validation**: `validate-ruleset --all` now reports **10/10 PASS in both repos** (8 prior + stewpot + trophy-dark).
- **For Intern J's dice tray**: `diceTagFormat.example` strings in both ruleset.json files document the exact chat-tag shape — `mrr-roll:` prefix universally since ruleset.json is mirror-identical; the namespace difference lives in the bundle.json layer when bundles are authored later.
- **Schema compatibility**: both rulesets explicitly satisfy the Round 8 tightened "exactly one under + exactly one over" invariant. Trophy Dark uses `Risk` and Stewpot uses `Capability` for the stat name — proves the schema treats stat names as opaque strings.
- **Not shipped this cycle** (deferred bundle artifacts): bundle.json, lorebook.json, gm-agent.md, INSTALL.md for both rulesets. Trophy Dark's gm-agent.md particularly important for the dark-overrule narration instructions since the schema can't enforce it.

### Tightened — `stance-modal-pool` schema directional invariant + L&F sample characters + INSTALL.md (2026-05-10)

Schema invariant now enforced at validation time + L&F bundle polish.

- **`schema/ruleset.schema.json`** — tightened the `stance-modal-pool` resolution branch with an `allOf` containing two `contains/minContains: 1/maxContains: 1` constraints on the `stances` array: one stance must have `direction: "under"`, the other `"over"`. The order is NOT constrained — players can author lasers-first or feelings-first. **Two-under or two-over rulesets now FAIL validation** with diagnostic `must contain at least 1 and no more than 1 valid item(s)`. The `description` updated to drop the old "enforced by the dice widget" wording. Existing L&F continues to validate.
- **NEW `rulesets/lasers-and-feelings/INSTALL.md`** (72 lines, both repos byte-identical): how to install the L&F bundle into Marinara, what stance-modal-pool means at the table, LASER FEELINGS exact-match rule, outcome tier interpretation. Namespace-neutral wording so the file is byte-identical across repos (the roll-tag prefix difference documented inline).
- **NEW `rulesets/lasers-and-feelings/characters/sample-pilot.json`** (46 lines, both repos byte-identical): "Sparks McGee" — Hot-Shot Pilot, Alien style, Number=4, Goal: "Become Captain". Hybrid envelope (`name/ruleset/role/style/number/goal/summary` + a `sheet` block mirroring `blankSheet()` shape from the renderer).
- **NEW `rulesets/lasers-and-feelings/characters/sample-doctor.json`** (46 lines, both repos byte-identical): "Doc Counterpart" — Heroic Doctor, Android style, Number=3, Goal: "Solve the mystery of the alien ruins".
- **Synthetic two-under negative test confirmed**: building a scratch ruleset with both stances `direction: "under"` produces a validate-ruleset FAIL with the expected `contains` diagnostic. Scratch deleted after verification.
- **Hand-off — bundle artifacts side observation**: Round 7 Intern S authored the L&F `bundle.json/gm-agent.md/lorebook.json/ruleset.json` files but they remained untracked at session end. Round 8 commit will pick them up alongside these new additions.

### Added — `quickRollFor*` STANCE branch + `applyDiceContextSpecialties` UNDER+STANCE branches (2026-05-10)

Completes the dice-tray dispatch story for both Round 6 (`roll-under`) and Round 7 (`stance-modal-pool`).

- **`extension/RPG-Extension-RP-Mode.js`** — extended four helpers with new mode branches:
  - `quickRollForSave` (lines 5636 guard, 5646-5668 branch): MODES.STANCE branch reads `state.ruleset.resolution.stat` → `state.sheet.attributes[<stat>]` (fallback `resolution.statDefault || 4`), opens the widget with `setDiceInput("stat", N)` and `setDiceInput("pool", 1)`. Does NOT auto-select stance — the LASERS/FEELINGS choice stays with the player.
  - `quickRollForDerived` (lines 5711 guard, 5713-5731 branch): same shape; ignores `derived.rollFormula` (not relevant under stance-modal-pool because every roll resolves against the single stat).
  - `quickRollForSkill` (lines 5985-6005 branch): preserves existing `state.diceContext = { skillName, base: {} }` initializer; sets `base.stat` + `base.pool = 1` and fires the same two `setDiceInput` calls. `renderSpecialtiesPane(skill)` still runs after the branch (no-op for L&F since skills have no specialties; future-proof for hypothetical stance rulesets that declare them).
  - `applyDiceContextSpecialties` (lines 6094-6113): UNDER branch sums VALUE-kind specialty values into the widget's `bonus` input (which `rollRollUnder` adds to `baseTarget` — the mental-model flip already wired in Round 7's quickRollFor); STANCE branch is an explicit no-op `return` with comment explaining why (L&F has no specialty-bearing skills; future hook documented).
- **Mirror discipline**: same in `Marinara-RPG-Extension` with `mrr-` namespace at corresponding lines (5507/5514-5536/5576-5596/5851-5871/5960-5979).
- **Validation**: `node --check` clean on both .js files. Existing SINGLE/POOL/FATE branches unchanged — no regression.
- **Future hook documented**: if a future ruleset declares both `resolution.mode === "stance-modal-pool"` and `skillSpecialties.enabled === true`, the natural interpretation would be `specialty.value → pool size adjustment` (each specialty tick adds a die).

### Added — `stance-modal-pool` resolution mode + Lasers & Feelings ruleset (2026-05-10)

New 6th branch in the schema's `resolution.oneOf` union — `stance-modal-pool` — supports hybrid roll-under/roll-over pool systems with optional exact-match special outcomes (the LASER FEELINGS bridge). Designed for indie one-pagers like L&F, BRP-family games, and the polarized-pool family generally (Trophy Dark light/dark dice, The Stewpot's action/stillness, etc.). Plan source: `~/cc-wiki/Roadmap/marinara-lasers-and-feelings-stance-modal-pool.md` (378-line spec).

- **`schema/ruleset.schema.json`** — added `stance-modal-pool` `oneOf` branch (lines 233-344, 112 lines) with required `mode`/`diceType`/`poolFormula`/`stat`/`stances`/`outcomeTiers` plus optional `exactMatch`. Inline sub-schemas for `stances` (exactly 2 items, per-item `direction` enum `under|over`), `exactMatch` (with `narrationHook` pattern), `outcomeTiers` (≥2 items, monotonic `minSuccesses`).
- **`extension/RPG-Extension-RP-Mode.js`** — `MODES.STANCE = "stance-modal-pool"` constant (line 69); dispatcher branch at line 8770; lines 8960-9145 implement `buildStanceModalPoolWidget` (segmented stance toggle + pool size + stat inputs + poolFormula hint) and `rollStanceModalPool` (strict under/over comparisons making double-count structurally impossible, exact-match counting, outcome-tier walk, `[mrrp-roll: ruleset=..., stance=..., stat=..., statValue=..., pool=..., dice=[...], successes=..., exactMatches=..., tier=..., narrationHook=...]` chat tag emission).
- **`extension/RPG-Extension-RP-Mode.css`** — new `/* ── Round 7: stance-modal-pool ── */` section at lines 3055-3171 with segmented stance toggle pill control, hint label, exact-match die outline ring, four tier-* result strip variants (miss/barely/good/critical).
- **NEW `rulesets/lasers-and-feelings/`** in both repos (mirror-identical content except namespace-required `schema` and `diceTagFormat` differences in bundle.json):
  - `ruleset.json` — single Number stat (2-5), LASERS (under) / FEELINGS (over) stances, LASER FEELINGS exact-match (counts as success + grants player a question), four outcome tiers (miss / barely / good / critical). One "Help" skill captures the helping-roll mechanic to satisfy the schema's `skills.minItems: 1` constraint.
  - `bundle.json` — minimal manifest
  - `lorebook.json` — Doubleclicks tribute setting (the Consortium, the Raptor, a starship of cool nerds) + four "create a space adventure" prep tables
  - `gm-agent.md` — stance-prompting, tier narration, LASER FEELINGS handling
- **Mirror discipline**: same in `Marinara-RPG-Extension` with `mrr-` namespace (schema at lines 233-344, JS at lines 69/8676/8866-9051, CSS at lines 3070-3186, ruleset/bundle/lorebook/gm-agent.md byte-identical mirrors).
- **Logic tests verified** (5/5 PASS via standalone Node harness using the same algorithm as `rollStanceModalPool`):
  - Lasers stance, pool=3d6, stat=4, faces=[1,4,6] → stance-successes=1, exactMatches=1, total=2, tier="good"
  - Feelings stance, same input → stance-successes=1, exactMatches=1, total=2, tier="good"
  - Exact-match no-double-count, faces=[4,4,4] stat=4 → stanceSuccesses=0, exactMatches=3, total=3, tier="critical"
  - Miss, faces=[5,6,6] Lasers stat=4 → all 0 → tier="miss"
  - Open-ended top tier, faces=[1,2,3,4] Lasers stat=5 → 4 successes → tier="critical"
- **`validate-ruleset --all` reports 8/8 PASS** in both repos.
- **Deviations from spec**: (1) Schema enforces exactly 2 stances via `minItems/maxItems: 2` but does NOT enforce one-under + one-over pairing — that would require nested conditional `oneOf` on items order; the dice widget treats equal-direction stances as a soft failure (FEELINGS stance with both stances under would just produce 0 successes for dice rolling above stat); flagged for Round 8 strict-schema follow-up. (2) Pool input is user-typed each roll with poolFormula shown as hint label, rather than auto-parsing `(prepared ? 1 : 0)` conditional syntax — matches how L&F plays at the table. (3) L&F gets a single "Help" skill to satisfy schema `skills.minItems: 1` without modifying the schema constraint.
- **Hand-off — Round 8 follow-ups documented in Intern S's hand-off notes**: extend `quickRollFor*` to handle stance-modal-pool, add a directional schema invariant, wire inline help bonus, ship sheet v0.2 (shared-state primitives), Trophy Dark / Stewpot generalizations, sample characters.

### Added — Call of Cthulhu 7e + GURPS-lite rulesets (roll-under exemplars) (2026-05-10)

Two reference rulesets that exercise the Round 6 `roll-under` mode end-to-end.

- **NEW `rulesets/coc7e/ruleset.json`** in both repos (308 lines, byte-identical):
  - Resolution: `roll-under` / `1d100` / `criticalSuccessFormula: "{target}/5"` / `criticalFailureThreshold: 96`
  - 8 attributes (STR, CON, SIZ, DEX, APP, INT, POW, EDU; range 15-99, default 50)
  - 46 skills with per-skill defaults (Spot Hidden 25, Dodge linkedAttribute Dex, Cthulhu Mythos 0, Fighting Brawl 25, Firearms specialties, etc.)
  - 6 derived stats: Hit Points (bar, `({CON}+{SIZ})/10`), Magic Points (bar, `{POW}/5`), Sanity (bar, starts at POW), Move (default 8 with conditional rule documented in `formula`), Luck (default 50), Build (default 0 with lookup table in `formula`)
  - 2 state blocks: Sanity (Stable/Temporary/Indefinite/Permanently Insane) and Status (Healthy/Wounded/Dying/Unconscious/Dead)
  - 4 resources cluster (HP bar, MP bar, Sanity counter, Luck counter)
  - Difficulties block (Regular 100, Hard 50, Extreme 20) — required by schema's `difficulties.minProperties: 2`; documentation-only for roll-under
  - `header.raceLabel: "Occupation"`, `header.classLabel: "Archetype"`
  - NO `morality`, NO `xpTable`, NO `skillProficiency`, NO `saves`, NO `abilities`, NO `conditions[]`
- **NEW `rulesets/gurps-lite/ruleset.json`** in both repos (247 lines, byte-identical):
  - Resolution: `roll-under` / `3d6` / `criticalSuccessFormula: "4"` / `criticalFailureFormula: "{target} + 10"` (MVP simplified from the full RAW)
  - 4 attributes (ST, DX, IQ, HT; range 1-25, default 10)
  - 24 skills with `linkedAttribute` per skill, defaults reflecting GURPS-lite default-from-attribute math (DX-5 for easy, DX for hard, etc.)
  - 7 derived stats: HP (`=ST`), FP (`=HT`), Will (`=IQ`), Perception (`=IQ`), Basic Speed (`(DX+HT)/4`), Basic Move (floor of Speed), Dodge (Speed+3)
  - 3 state blocks: Status (6 values), Fatigue (4 values), Posture (5 values)
  - 2 resources cluster (HP bar with `max: {ST}`, FP bar with `max: {HT}`)
  - Difficulties block (Easy/Average/Hard/VeryHard with canonical skill default offsets 0/-1/-2/-3)
  - `header.raceLabel: "Race"`, `header.classLabel: "Template"`
  - NO `morality`, NO `xpTable`, NO `skillProficiency`, NO `saves`, NO `abilities`
- **Validation**: `validate-ruleset --all` reports 7/7 PASS in each repo (5 pre-existing + CoC + GURPS) at the moment of Intern X's commit; combined with Intern S's L&F, the total post-Round-7 is **8/8 PASS in both repos**.
- **Deviations from spec**:
  - CoC Move/Luck/Build derived stats encoded with static defaults + plain-language `formula` documentation because the conditional / lookup-table math isn't expressible in the schema's arithmetic-only whitelist (`{Stat}` substitution + + - * /). Future schema evolution could add conditional formula support.
  - CoC `abilities` block omitted — Mythos spells are open-ended and don't map cleanly to `abilities.categories[]`. Spell content deferred to a future content cycle.
  - GURPS-lite `Observation/Survival/Tracking` use `linkedAttribute: "Perception"` (derived stat) — RAW for GURPS but assumes the sheet renderer resolves derived-stat linkage; falls back to "Intelligence" if not.
  - GURPS-lite `criticalFailureFormula` uses the simpler `{target} + 10` (not the full RAW natural-17/18-OR-target+10) per MVP framing.
  - Both bundles omit `equipmentSlots`, `equipmentBonusTargets`, sample characters, and `gm-agent.md` — all deferred to a content cycle.

### Added — `quickRollFor*` helpers wired for roll-under mode (2026-05-10)

Extends Round 6's roll-under to actually work from the sheet — previously, clicking the Roll button on a skill/save/derived row in a roll-under ruleset did nothing because the helpers bailed out on `mode !== MODES.SINGLE`.

- **`extension/RPG-Extension-RP-Mode.js`** — extended three helpers to handle `MODES.UNDER`:
  - `quickRollForSave` (lines 5633-5681): guard widened to `SINGLE|UNDER`; advantage check gated under SINGLE; new UNDER branch pre-fills target from `state.sheet.skills[save.name]` (fallback to linked attribute) + bonus from `equippedBonuses(save.name).value`.
  - `quickRollForDerived` (lines 5686-5718): guard widened; new UNDER branch evaluates `derived.formula` for target + `equippedBonuses(derived.name).value` for bonus.
  - `quickRollForSkill` (lines 5881-5946): new `else if (mode === MODES.UNDER)` branch after FATE; pre-fills target from `state.sheet.skills[skill.name]`, bonus from `equippedBonuses(skill.name).value + tierBonus`.
- **Bonus direction correctly inverted** for UNDER mode: bonuses populate the widget's `bonus` input, which `rollRollUnder` adds to `baseTarget` to produce the effective target (`var target = baseTarget + bonus;`). Result line prints e.g. `(target 65+5)` so the math is visible to the player.
- **For `quickRollForSkill`**, also bundles `tierBonus` (proficiency-tier bonus) into the bonus input since skill proficiency raises the cap in roll-under (Trained CoC skill is effectively higher than the raw stored value).
- **SINGLE-mode behavior unchanged** — UNDER is a strictly-additive branch in each helper; existing rulesets (D&D, PF2e, Exalted) keep working.
- **Mirror discipline**: same in `Marinara-RPG-Extension` (RP namespace: identical function-level identifiers; only DOM ids/classes use `mrr-` prefix).
- **Out of scope this cycle (Round 8 follow-ups)**: `applyDiceContextSpecialties` (the skill-specialty checkbox recomputer) still only branches on POOL/SINGLE/FATE — UNDER specialty support is a follow-up. Once stance-modal-pool's widget stabilizes, `quickRollFor*` should also gain a `MODES.STANCE` branch (pre-fill stance toggle + pool size from sheet state).

### Added — Roll-under resolution mode (system-neutral percentile + stat-vs-XdY) (2026-05-10)

New `resolution.mode: "roll-under"` for percentile (Call of Cthulhu, BRP, Runequest, Pendragon) and stat-vs-XdY (GURPS, traditional BRP) systems. Bonuses contribute to the TARGET (raise the cap), not to the dice roll — opposite of `single-roll` where bonuses lift the roll total.

- **`schema/ruleset.schema.json`** — added a `roll-under` `oneOf` branch on `resolution` with required `diceFormula` (XdY regex `^[1-9][0-9]*d[1-9][0-9]*$`) and optional `skillBonusFormula`, `criticalSuccessFormula`, `criticalFailureThreshold` (integer ≥1), `criticalFailureFormula`. `additionalProperties: false` enforced.
- **`extension/RPG-Extension-RP-Mode.js`** — `MODES.UNDER = "roll-under"` constant (line 68); dispatcher branch at line 8709 (`else if (mode === MODES.UNDER) buildRollUnderWidget();`); four new functions at lines 8809-8907: `buildRollUnderWidget` (dice tray Target + Bonus input + Roll button), `parseRollUnderFormula` (XdY regex parser), `evalRollUnderFormula` (sandboxed arithmetic-only Function-eval with whitelist regex, supports `{target}` and `{margin}` token substitution), `rollRollUnder` (rolls dice, sums total, compares `total ≤ target` for success, applies crit-success when `total ≤ criticalSuccessFormula(target)` and fumble when `total ≥ criticalFailureThreshold` or per `criticalFailureFormula`).
- **`extension/RPG-Extension-RP-Mode.css`** — new `/* ── Round 6: roll-under mechanic ── */` section at lines 3041-3052: `.mrrp-dice__result--crit` (accent ring) and `.mrrp-dice__result--fumble` (warning fill) modifiers reusing existing tokens (no new tokens introduced).
- **Mirror discipline**: same in `Marinara-RPG-Extension` with `mrr-` namespace (schema mirror identical, JS at lines 68/8620/8719-8817, CSS at lines 3056-3067).
- **Embedded CSS regenerated** — `node tools/embed-css.mjs` re-ran in both repos after Round 6: RP 80447 chars / GM 79523 chars.
- **Validation**: 15-case logic test passed (CoC 7e + GURPS + plain + bonus-target + injection-rejection); `validate-ruleset --all` still 5/5 PASS in both repos (the new enum value is permissive — existing rulesets don't have to use it).
- **What's needed to actually USE roll-under**: a `coc7e` or `gurps-lite` ruleset bundle that declares `resolution: { mode: "roll-under", diceFormula: "1d100"|"3d6", criticalSuccessFormula: "{target}/5"|"4", criticalFailureThreshold: 96|null }`. Plus quickRoll helpers wiring to populate the target input automatically from skill values, and route equipped-bonuses to the target (mental-model flip from single-roll). Both deferred to a future cycle.
- **Out of scope this cycle**: hybrid-resolution rulesets that mix roll-under for some skills and roll-over for others (would need sub-mode-per-skill or mixed-resolution mode design — deferred per roadmap).

### Removed — Orphaned legacy-identity CSS comment block (2026-05-10)

Round 4 deleted the legacy `renderIdentityField` function and `.mrr(p)-sheet__id-*` CSS rules but left behind a 4-line comment block at lines 153-156 of each `extension/*.css` file that described the now-deleted rules. This pass removes that orphan plus its trailing blank line (5 lines total per repo) to preserve single-blank-line separation between the preceding `.mrr(p)-sheet__char-row` rule and the active Phase 5 step 5.1 identity-card comment.

- **`extension/RPG-Extension-RP-Mode.css`** — deleted lines 153-157 (4 orphan-comment lines + 1 trailing blank).
- **`extension/RPG-Extension-GM-Mode.css`** — same deletion with `mrr-` namespace surrounding context.
- **Mirror discipline**: identical 5-line deletion in both repos.
- **Pre-deletion verification**: confirmed the deleted comment described only the now-removed legacy identity-row rules; the adjacent Phase 5 step 5.1 comment block (now at line 153, shifted up from 158) was left intact verbatim.
- **Embedded CSS regenerated** in the same Round 6 post-agents pass.

### Added — Pathfinder 2e content enrichment: tooltipFormula, condition descriptions, recovery clauses (2026-05-10)

Round 5 (step B.5) created the PF2e ruleset as a schema-conforming MVP. This pass enriches PF2e content with 15 substantive additions that don't require schema changes.

- **`rulesets/pathfinder2e/ruleset.json`** — grew from 721 to 732 lines:
  - **`derivedStats[].tooltipFormula`** (1 addition): Hit Points now declares `8 + ((10 + {Constitution_mod}) * {Level}) + {bonuses:Hit Points}` as a Fighter-representative formula (ancestry 8 + class 10/level + CON_mod/level baseline). The existing `formula` field retains the class-agnostic GM-readable plain-language version, documenting that class HP/level legitimately ranges 6 (Wizard) to 12 (Barbarian).
  - **`states[]` condition descriptions** (10 enrichments): Off-Guard, Frightened, Sickened, Stunned, Prone, Wounded, Dying, Doomed, Drained, Persistent Damage now have expanded canon descriptions (graduated 1-4 mechanics, recovery rules, action cost, interaction rules) and new `trigger` fields.
  - **Top-level `conditions[]` refinements** (4 enrichments): Off-Guard, Drained, Stupefied, Fascinated descriptions expanded with canon detail (renamed-from-flat-footed note, graduated effects, flat-check spell-fizzle on Stupefied, Will-save recovery clause on Fascinated).
- **`xpTable[]` verified**: levels 1-20 thresholds confirmed correct against PF2e Core Rulebook canon. No adjustments needed.
- **Mirror discipline**: byte-identical changes in `Marinara-RPG-Extension`.
- **Validation**: `node tools/validate-ruleset.mjs --all` still 5/5 PASS in both repos.
- **Schema-blocked items deferred** (would need schema evolution): `resources[].description` for Hero Points / Focus Points context, Lore subspecialization pattern (PF2e Lore is one trained skill per topic), per-class `hpPerLevel` table for true class-agnostic Hit Points formula.
- **Future PF2e content surfaces** (not addressed this cycle): the remaining 11 `states[]` conditions still need full trigger fields (Quickened, Slowed, Grabbed, Restrained, Confused, Fascinated [partial], Enfeebled, Stupefied [partial], Clumsy, Unconscious + 1 more); spell slots beyond rank 1-3 (4-10 + cantrips); weapon/armor/feat content (outside ruleset.json by current design).

### Added — Phase 5 step 5.6 (V:TM V20 subset): Morality section + path picker + virtue toggles + `v20-health-track` renderer (2026-05-10)

V20 visual treatment for the morality block (Plan B step 1 + B.3 schema/data) and the custom `v20-health-track` renderer that Round 4's Resources cluster (step 5.3) called for. Closes the V20-specific surface area of step 5.6.

- **`extension/RPG-Extension-RP-Mode.css` end-of-file** — new `/* ── Phase 5 step 5.6: V20 morality + paths + virtues + health-track ── */` section (lines 2796-3044, +249 lines): `.mrrp-morality` cluster (Path Rating numeric stepper, `__path-picker` dropdown, `__path-desc` 1-2 line description below picker, `__virtue-row` with paired-choice segmented control or single-label variant), `.mrrp-health-track` grid (7 levels × box-per-level, click to cycle empty → B → L → A → empty, severity-colored per damage type), narrow-panel media query for ≥320px responsiveness.
- **`extension/RPG-Extension-RP-Mode.js`** — single-line section dispatch wiring at line 3747 (`else if (sec === "morality") mrrpP3RenderMoralitySection(state.mountEl)`). Lines 11320-11750 (+431 lines): `mrrpP3RenderMoralitySection`, `mrrpRenderVirtueRow` (handles `options[]` paired-choice and non-options single-virtue cases), `mrrpRenderPathPicker` (lists `morality.paths[]`, on-change writes `state.sheet.morality.path` and auto-resets paired-choice virtue actives to the new path's canonical defaults), `mrrp_v20PathVirtueMap` (path → {description, canonical virtue actives} table; mirrors lorebook `Path Lookup:` entries verbatim because lorebook entries are Marinara-server-side and not loadable from the extension at render time), plus the `v20-health-track` registration in `mrrp_resourceRenderers` (renders the 7-level damage-typed track; stores `state.sheet.resources["health"].track = [{type: "B"|"L"|"A"|null}, ...]`; writes `entry.current = levels.length - damaged` so existing `mrrpGetResourceCurrent`-based code resolves to a sane integer).
- **Mirror discipline**: same in `Marinara-RPG-Extension` with `mrr-` namespace (CSS at line 2811-3059, JS dispatch at line 3563, JS implementation at line 11227-11657).
- **Embedded CSS regenerated** — `node tools/embed-css.mjs` re-ran in both repos after Round 5: RP grew to 80182 chars / GM to 79276 chars (+6.6K each from the V20 5.6 CSS).
- **V20 Health resource** now renders the actual track UI in both repos — previously showed the placeholder "Custom component v20-health-track not registered" because the registry was empty. The registry-default `{current: number}` shape didn't fit V20 per-level damage typing, so the renderer owns its own state shape under its resource id and shadow-writes `current` for compatibility.
- **Path-virtue mapping** (per V20 RAW + lorebook): Humanity → Conscience + Self-Control; Path of Honorable Accord → Conviction + Instinct; Path of Caine → Conviction + Instinct; Path of Beast → Conviction + Instinct; Path of Night → Conviction + Instinct ("Cold variant" alternative for Self-Control documented in lorebook). User can override per-virtue after selecting the path; overrides persist until path changes again.
- **Hand-off note**: future dice integrations should read the active option (e.g., `state.sheet.morality.virtues["self-control-instinct"].active`) when picking which virtue to roll — the active label matters, not the row id.
- **Lorebook-fetch deviation**: spec called for runtime lorebook lookup via `loreRef`. Lorebook entries are server-side at install time, not loadable from the extension at render time. Used an embedded `mrrp_v20PathVirtueMap` with descriptions verbatim from `rulesets/vtmv20/lorebook.json` `Path Lookup:` entries instead. Same data, kept in sync because the rulesets dir is the source of truth for the lorebook install. When a server-side fetch path lands, the map becomes redundant and the renderer code already only reads `pathMap.description`.

### Added — Density-aware threading: existing components now respond to the density toggle (2026-05-10)

Follow-up to Phase 5 step 5.5 (Round 2). The density toggle was previously declared with `[data-density]` branches but most existing components didn't reference the `--mrr(p)-density-*` variables, so the toggle had no visible effect on most of the sheet. This pass threads density vars through 13 existing component rules per repo.

- **`extension/RPG-Extension-RP-Mode.css`** — 13 existing component rules modified (17 individual literal-to-density-var replacements) at lines 99, 119, 291, 326, 784, 834, 1051, 1329, 1738, 1775, 1812, 1821, 1897, 2243, 2282. Categories:
  - **padding (full pad-x + pad-y)** — 3 rules: `.mrrp-section`, `.mrrp-p3-section__head`, `.mrrp-p3-panel__body`
  - **gap** — 3 rules: `.mrrp-sheet__header`, `.mrrp-section`, `.mrrp-row`
  - **font-size (body text)** — 7 rules: `.mrrp-sheet` (root cascade), `.mrrp-dice`, `.mrrp-dice__input`, `.mrrp-dialog__lib-name`, `.mrrp-spellbook` / dialog panel, `.mrrp-p3-row__name`, `.mrrp-p3-row__val--auto`
  - **row-height** — 2 rules: `.mrrp-p3-row` height, `.mrrp-p3-row--skill` min-height
  - **pad-x-only / pad-y-only (asymmetric)** — 2 rules: `.mrrp-p3-section__body` (preserves 8px top + density x/y), `.mrrp-p3-panel__head` (preserves 10px top + density x)
- **Mirror discipline**: same 13 rules in `Marinara-RPG-Extension` with `mrr-density-*` namespace.
- **Components that visibly respond to density now** (sampled): every `.mrr(p)-section` card, both row primitives (`.mrr(p)-row` legacy + `.mrr(p)-p3-row` current), Phase-3 section heads and panel bodies, the dice tray, and dialog/spellbook panels. Switching `[data-density]` between compact/cozy/roomy now visibly affects a substantially larger fraction of the sheet.
- **Components left UNTHREADED on purpose** (with rationale):
  - **Type-scale labels** (9px / 10px / 11px / 16px uppercase letter-spaced) — type-scale decisions, not density-driven.
  - **Tiny pill button paddings** (`.mrr(p)-char-btn`, `.mrr(p)-density-toggle__btn`, `.mrr(p)-p3-row__roll`) with `padding: 2px 6px;` — control-sizing decisions; threading would oversize chips at higher density.
  - **Stepper button dimensions** (22×22 with 13px glyph) — control-sizing, not density.
  - **Border widths, border-radius, shadow offsets, line-height** — visual identity, not density.
  - **Inter-card vertical rhythm** (`margin-bottom: 8px` on `.mrr(p)-p3-section`) — layout-rhythm decision distinct from intra-component density.
- **Phase 5 step 5.1 identity card** still uses Phase 4 tokens `--mrr(p)-pad`/`--mrr(p)-gap` (left untouched per ownership boundary — Phase 5.1 section is owned territory). A future cycle can evolve identity-card density-awareness inside that section.
- **Embedded CSS regenerated** in the same Round 5 post-agents pass (orchestrator-owned single `embed-css.mjs` run).
- **JS-side density awareness flagged for separate cycle** — any JS code that constructs inline style strings (e.g. `el.style.fontSize = '13px'`) bypasses the density vars. A grep over `extension/*.js` for inline px literals in padding/gap/height/font-size categories would find any remaining un-threaded surfaces. Not addressed in this cycle.

### Added — Plan B step B.5: `rulesets/pathfinder2e/ruleset.json` (creation — closes the validate-rulesets ENOENT gap) (2026-05-10)

PF2e's `ruleset.json` was previously missing (validate-rulesets reported ENOENT for both repos). This step creates the file as a schema-conforming MVP for Pathfinder 2nd Edition.

- **NEW file `rulesets/pathfinder2e/ruleset.json`** in both repos (721 lines, 22194 bytes, byte-identical mirror):
  - **6 attributes**: STR, DEX, CON, INT, WIS, CHA — using `modifierFormula: "{Score}"` because PF2e stores raw modifiers (Remaster removed ability scores), not 1-30 score values.
  - **17 skills**: Acrobatics, Arcana, Athletics, Crafting, Deception, Diplomacy, Intimidation, Lore, Medicine, Nature, Occultism, Performance, Religion, Society, Stealth, Survival, Thievery.
  - **3 saves**: Fortitude (CON), Reflex (DEX), Will (WIS).
  - **`skillProficiency.tiers`**: 5-tier UTEML (Untrained / Trained / Expert / Master / Legendary) with Level-scaled rollBonusFormula `{Level} + 0/2/4/6/8`.
  - **10 derivedStats**, **8 with `tooltipFormula`**: Armor Class, Class DC, Spell DC, Perception, Fortitude/Reflex/Will Save, Speed (with formula breakdowns). Hit Points and Level are plain.
  - **6 resources**: `hit-points` (bar, color bad), `hero-points` (counter, max 3), `focus-points` (counter, default 0 for casters with focus spells), `spell-slots-1` / `-2` / `-3` (counter, grouped under "Spell Slots").
  - **21 conditions** in `states[]` (single "Condition" state group): includes spec-required Frightened, Sickened, Drained, Wounded, Dying, plus Off-Guard, Stunned, Doomed, Persistent Damage and 11 others.
  - **`xpTable[]`** with 20 entries (levels 1-20, +1000 XP per level after level 1, per PF2e Core Rulebook).
  - **`sections.order[]`** (12 entries): identity, xp, resources, attributes, saves, skills, derived, abilities, states, conditions, inventory, notes.
  - **`resolution.mode: "single-roll"`** with full formula set (modifier / skill / spellSaveDc / attackProficiency).
  - **`morality` NOT declared** — N/A for PF2e.
- **Header fields**: PF2e wants Class + Heritage + Background + Ancestry + Level (5-field case flagged in roadmap "Open questions"). Since the schema currently exposes only `header.raceLabel` / `header.classLabel` for two free-text slots, declared `raceLabel: "Ancestry"` / `classLabel: "Class"`; Heritage and Background remain GM-agent-prompted free text (likely captured in notes). Adding a real `identityFields[]` is a schema-evolution concern for Plan C.
- **Validation gates**: `node tools/validate-ruleset.mjs --all` now reports **5/5 PASS in both repos** — the PF2e ENOENT failure that has been flagged since Phase 4 is closed.
- **PF2e mechanics intentionally NOT encoded** (out of MVP scope): spellcasting class variants, animal companions / familiars / eidolons, archetypes and dedication feats, ancestry heritages, class-specific resources (Rage points, Composition Points, Mythic Points), critical specialization effects per weapon group, MAP automation, three-action economy enforcement, sub-systems (Influence, Research, Chase, Hexploration, Victory Points).
- **For future PF2e content cycle**: bundle.json already has rich lorebook entries for Investiture, Hero Points, MAP, Recall Knowledge, XP awards, and degrees of success. Future cycles should encode class-specific focus pools, archetype dedications, and the spellcasting variant matrix. Note bundle.json has divergent `mrr-`/`mrrp-` namespaces — that pre-existing inter-repo difference was NOT touched this cycle.

### Added — Phase 5 step 5.3: Resources cluster ("charbar") (2026-05-10)

The biggest Phase 5 piece — a horizontal resource readout cluster above Attributes, driven by `ruleset.resources[]` (Plan B step 1 schema add). Five sub-renderers dispatch by `type`: bar / dice / counter / pool / custom. Includes a custom-component registry that Round 5's V20 visual treatment will populate.

- **`extension/RPG-Extension-RP-Mode.css` end-of-file** — new `/* ── Phase 5 step 5.3: Resources cluster ── */` section (lines 2552-2794, +243 lines) with sub-renderer styles for `.mrrp-resources` cluster, `.mrrp-resource__bar` (auto-color fill: <30% bad, 30-65% warn, >65% ok), `.mrrp-resource__dice` (clickable die-glyph pool), `.mrrp-resource__counter` (numeric stepper), `.mrrp-resource__pool` (current/max + quickButtons), `.mrrp-resource__placeholder` (custom-component fallback), `.mrrp-resources__group-label` (subheader for grouped resources), and `flex-wrap` with `min-width: 120px` per resource for narrow-panel responsiveness (≥320px collapses to single column).
- **`extension/RPG-Extension-RP-Mode.js`** — registry `mrrp_resourceRenderers = {}` (open-object for runtime registration); helpers `mrrpResolveResourceMax`, `mrrpResolveResourceDefaultCurrent`, `mrrpGetResourceCurrent`, `mrrpSetResourceCurrent`, `mrrpResourceClamp`, `mrrpResourceAutoColor`, `mrrpResourceContext` (wraps `statContext()` to add `{Level}` and `{tierBonus}` token resolution that the existing context omits); five sub-renderers (`mrrpRenderResourceBar/Dice/Counter/Pool/Custom`); custom dispatcher `mrrpRenderResourceCustom` that looks up `rendererConfig.component` in the registry and falls back to `mrrpRenderResourcePlaceholder` on miss; section function `mrrpP3RenderResourcesSection`. Section dispatch wired with a new `sections.order[]` fallback chain (`sheetSections[]` → `sections.order[]` → hardcoded default) and a `"resources"` case that calls the new section function. Lines 3723-3759 (dispatch wiring) + lines 3818-4209 (registry, sub-renderers, dispatcher).
- **State persistence** at `state.sheet.resources[<id>].current`. quickButtons handle integer deltas and the `"max"` magic string. `change` event (not `input`) on numeric editors prevents bar-fill stutter while typing multi-digit numbers.
- **Group clustering** works for D&D Spell Slots case: adjacent resources sharing the same `group` value cluster under one subheader (`.mrrp-resources__group-label` with `flex-basis: 100%` to force wrap break).
- **Mirror discipline**: same in `Marinara-RPG-Extension` with `mrr-` namespace (CSS at line 2567-2809, JS dispatch at line 3533-3576, JS renderers at line 3637-4035).
- **Embedded CSS regenerated** — `node tools/embed-css.mjs` re-ran in both repos after Round 4 (Intern A additions + Intern B deletions): RP grew to 73594 chars / GM to 72786 chars.
- **Hand-off — V20 health-track registration (Round 5):**
  ```js
  // RP file:
  mrrp_resourceRenderers["v20-health-track"] = function (resource, parent, ctx) { /* ... */ };
  // GM file: same code, swap mrrp_ → mrr_
  ```
  The renderer owns its own state shape under its resource id (typed `[{type:"B"|"L"|"A"|null}, ...]` array per V20's per-level damage typing) — the registry-default `{current: number}` shape doesn't fit V20 health.
- **Cross-track note**: this CSS change combined with Intern B's dead-code deletion in the same Round 4 was made parallel-safe via targeted `Edit` calls operating on disjoint regions. Both edits coexisted; embed-css.mjs ran ONCE post-agents to sync both into `EMBEDDED_CSS`.

### Removed — Phase 5 step 5.1 dead code: legacy `renderIdentityField` + `.mrr(p)-sheet__id-*` CSS rules (2026-05-10)

Cleanup follow-up to Phase 5 step 5.1 (Round 1) — removes the legacy identity rendering code that was left in place during the original cycle and is now confirmed unused.

- **`extension/RPG-Extension-RP-Mode.css`** — deleted lines 157-184 (baseline numbering): `.mrrp-sheet__identity-row`, `.mrrp-sheet__id-label`, `.mrrp-sheet__id-input`, `.mrrp-sheet__id-input:focus`. Net −28 lines.
- **`extension/RPG-Extension-RP-Mode.js`** — deleted lines 4816-4847 (baseline numbering): `renderIdentityField` function + 5-line preceding doc comment. Net −32 lines.
- **Pre-deletion verification** confirmed zero callers of `renderIdentityField` outside its own definition; zero references to deleted CSS classes from active JS code (only EMBEDDED_CSS string literal at line 49, which is regenerated automatically by `embed-css.mjs`).
- **Active code preserved**: 10 `.mrr(p)-identity*` rules in each CSS file remain in place; `mrr(p)RenderIdentitySubField` helper untouched. Verified via post-deletion grep.
- **Mirror discipline**: same deletions in `Marinara-RPG-Extension` with `mrr-` namespace.
- **Embedded CSS regenerated** in the same Round 4 post-agents pass; the `EMBEDDED_CSS` literal in both `.js` files no longer carries the deleted rules.
- **Tidy-up follow-up flagged**: an orphaned comment block at line 153 of each CSS file (now describing nothing, since it pointed to the deleted legacy identity-row rules) remains. Drop in a small follow-up cycle.

### Added — Plan B step B.4: fate-core schema-conforming stubs (2026-05-10)

Light JSON-only population of fate-core's new Plan B step 1 schema fields without inventing Fate Core mechanics that don't exist.

- **`rulesets/fate-core/ruleset.json`** — added a 38-line `sections` + `resources` block:
  - `sections.order[]` (9 entries): `identity, resources, attributes, skills, derived, abilities, states, inventory, notes`. Mirrors the existing legacy `sheetSections` order with the wider Plan B section vocabulary, inserts `resources` near top (matching the dnd5e/exalted3e pattern), and adds `identity`/`inventory`/`notes` defaults that were missing. The legacy `sheetSections` is preserved untouched for backward compatibility.
  - `sections.hidden[]`: explicit empty array — no fate-core sections need hiding.
  - `resources[]` (1 entry): `fate-points` with `type: "counter"`, max/current bound to `{Refresh}` (the Refresh attribute, default 3, max 5), `color: "accent"`, and `quickButtons: [-1, +1, Refresh]`.
- **Stress tracks intentionally NOT added to `resources[]`** — Physical Stress and Mental Stress are already represented as `derivedStats[]` entries with `renderAs: "track"`, which is the canonical Plan B representation for box-track stress. Promoting them to `resources[]` would duplicate the data and lose per-box penalty semantics (`track[].penalty`).
- **Fate Points exists in BOTH `derivedStats` AND `resources[]`** — the derivedStats version has `renderAs: "value"` (a single integer); the resources version has `type: "counter"` with quickButtons. The resources version is the better player-facing UX (top-of-sheet, click-to-spend) per Phase 5 step 5.3 intent. Coexistence is acceptable; deprecate the derivedStats version in a future cycle once resources rendering ships and migration is clear (renderer should prefer resources[] when both are present).
- **`derivedStats[].tooltipFormula`**: zero additions. Fate Core's derived stats are narrative formulas in plain English ("3 boxes baseline; Physique 1-2 grants box 3 already") that can't be expressed under the schema's whitelist of `digits, + - * / ( ) and {StatName}` because the rules use stepwise conditionals.
- **`abilities.groups[]`**: omitted entirely. Fate Core uses the player-chosen skill pyramid, not canonical groups like V20 Talents/Skills/Knowledges. Existing `abilities.categories: [{id:"stunts"}]` block untouched.
- **`morality`**: confirmed not added (N/A for Fate Core).
- **Mirror discipline**: byte-identical changes in `Marinara-RPG-Extension`.
- **Validation gates**: `node tools/validate-ruleset.mjs --all` PASS for fate-core in both repos (`fate-core v1.0.0`); mirror diff empty.
- **Forward-compatibility note**: if a `fate-accelerated` sibling ruleset is later added, its skill list becomes Approaches (Careful/Clever/Flashy/Forceful/Quick/Sneaky), Stress collapses to a single 3-box track, and most of these Plan B schema additions translate directly.

### Added — Phase 5 step 5.6 (Exalted-only): Initiative-Crashed badge + Anima Banner 6-value layout (2026-05-10)

Visual treatment for the Exalted content that Plan B step B.2 landed earlier today. Initiative state now has a visible distress treatment when active value is `Crashed`; Anima Banner accommodates the new `Suppressed` option with a muted styling.

- **`extension/RPG-Extension-RP-Mode.css` end-of-file** — new section `/* ── Phase 5 step 5.6: state badges + anima banner ── */` (~lines 2503-2578). Three new tokens scoped inside the section's local `:root` block: `--mrrp-state-crashed-color: oklch(0.62 0.18 28)`, `--mrrp-state-crashed-soft` (18% alpha), `--mrrp-state-crashed-line` (42% alpha). Selectors: `.mrrp-state--initiative-crashed` (BEM modifier on the existing `.mrrp-state` row) repaints `.__name`, `.__name::after` (appends `(!)` glyph), and `.__select` with red + soft red bg + red border. 2.4s ease-in-out opacity pulse 1→0.72→1, gated behind `prefers-reduced-motion`. `.mrrp-state--anima-suppressed` paints `var(--mrrp-text-faint)` + italic. Defensive `flex-wrap: wrap; min-width: 0;` on `.mrrp-state` and `max-width: 100%; min-width: 0;` on `.mrrp-state__select` to keep narrow-panel rows clean.
- **`extension/RPG-Extension-RP-Mode.js`** — new helper `mrrpStateRowApplyMods(row, name, value)` (~lines 4302-4326) idempotently strips any prior `mrrp-state--*` modifier and adds the right one for the active value: `--initiative-crashed` for Initiative=Crashed, `--anima-suppressed` for Anima Banner=Suppressed. Called once at row creation and re-called inside the select `change` handler so the visual updates live without rebuilding the section. Sets `data-active-value` on every state row for future CSS extension without growing the JS branch list.
- **Mirror discipline**: same in `Marinara-RPG-Extension` with `mrr-` namespace (CSS at line 2518-2593, JS helper at line 4146-4170, inline calls at 4197-4203).
- **Embedded CSS regenerated** — `node tools/embed-css.mjs` re-ran in both repos after Round 3's three agents (5.2 + 5.6 Exalted + Plan B B.3) all landed; both `EMBEDDED_CSS` strings now contain section markers for steps 5.1, 5.2, 5.4, 5.5, and 5.6.
- **Class-shape deviation from spec**: Used `.mrrp-state--initiative-crashed` (BEM modifier on existing `.mrrp-state` block) rather than `.mrrp-state-badge--initiative-crashed`, because the in-repo convention has no `.mrrp-state-badge` block — modifiers hang off the existing `.mrrp-state` row class. Functionally identical.
- **Anima Banner is select-based, not a pill strip.** Existing render uses native `<select>` so the option count never affects row width. Phase 5 step 5.6 ensures the new Suppressed option (Plan B B.2) renders correctly with muted treatment when active. If a horizontal pill-strip rebuild was the actual intent, that's a separate render-flow change for a follow-up cycle.

### Added — Phase 5 step 5.2: XP card visual restructure to prototype parity (2026-05-10)

Numeric input type-scale tightened to prototype spec (16px/600 for current/total/next, 18px/600/56px-wide for level), and the `+1 XP` button moved inline at the right end of the pool-mode row (prototype anatomy: `align-self: flex-end; margin-left: auto;`). Both formula-mode and pool-mode confirmed working.

- **`extension/RPG-Extension-RP-Mode.css` end-of-file** — new section `/* ── Phase 5 step 5.2: XP card ── */` (~lines 2460-2502) with five selector rules overriding type-scale and button position on the pre-existing `.mrrp-xp-card*` baseline. The baseline rules (lines 1615-1737, structural flex/padding/border/gap/group/bar geometry) were left untouched; the appended section overrides only the rules that diverged from the prototype. CSS source-order win resolves the cascade.
- **`extension/RPG-Extension-RP-Mode.js`** — single semantic change in `renderXpCard`'s pool branch (~lines 2746-2755): `+1 XP` button's parent argument changed from `card` → `poolRow` so it sits inline with the inputs instead of below them. Click handler still atomically increments both `state.sheet.xp.current` and `state.sheet.xp.total`. No state shape change, no schema change.
- **Pool-mode (Exalted, `resolution.mode: "dice-pool"`)**: renders `[CURRENT] / [TOTAL EARNED] [+1 XP]` all in one row.
- **Formula-mode (D&D 5e, `resolution.mode: "single-roll"`)**: renders `Level [N] · Current XP · / · Next-level threshold` row + 4px progress bar, reading thresholds from `ruleset.xpTable[]` (D&D 5e ships a 20-entry table; bar fill computes against current-level threshold).
- **Mirror discipline**: same in `Marinara-RPG-Extension` (CSS at line 2475-2517, JS at line 2461-2470).
- **Type scale matches prototype** per UI-build.md §3.4: 10-11px uppercase letter-spaced labels, 16px/600 numeric values, 18px/600 level input.
- **Card-label color preserved**: `var(--mrrp-accent)` retained rather than reverted to `var(--mrrp-text-faint)` as the prototype uses — keeps the in-extension stylistic convention where every section-label tints accent.

### Added — Plan B step B.3: V:TM V20 lorebook path-lookup entries (2026-05-10)

Plan B step 1 already populated the V20 morality block (paths, virtues with paired-choice options) and abilities.groups (Talents/Skills/Knowledges with verbatim 30-ability membership) more thoroughly than the original Plan B step B.3 brief expected. This step adds the missing piece: clean lookup-style lorebook entries that bind cleanly to the `loreRef` tokens declared on each path.

- **`rulesets/vtmv20/lorebook.json`** (untracked-but-modified file) — appended 5 new entries (~lines 425-466) named `Path Lookup: <loreRef>` to bind to the morality.paths[].loreRef tokens:
  - `Path Lookup: humanity` (binds Humanity)
  - `Path Lookup: path-honorable-accord` (binds Sabbat Lasombra/Tzimisce path)
  - `Path Lookup: path-caine` (binds Noddist scholars path)
  - `Path Lookup: path-beast` (binds frenzy-embracing predators path)
  - `Path Lookup: path-night` (binds Sabbat dark mystics path)
- Each new entry's `keys[0]` exactly matches the loreRef token from `morality.paths[N].loreRef`, enabling the future Phase 5 step 5.6 V20 path picker to resolve descriptions by direct lorebook lookup.
- **`rulesets/vtmv20/ruleset.json`** — no changes this cycle. Plan B step 1 already populated `morality.paths[]`, `morality.virtues[]` (with Conscience/Conviction and Self-Control/Instinct paired-choice options), `abilities.groups[]` (full 30-ability membership), `resources[]`, and `sections.hidden[]` for Disciplines.
- **Clan-specific Discipline trios verified** in lorebook clan entries (Brujah: Celerity/Potence/Presence; Toreador: Auspex/Celerity/Presence; etc.) — no gaps, no edits needed.
- **Mirror discipline**: byte-identical lorebook additions in both repos. Pre-existing line-420 `mrrp-state` vs `mrr-state` namespace difference inside content remains intact.
- **Hand-off — Round 4 V20 visual treatment** consumes these loreRef bindings: when the path picker selects a path, look up the lorebook entry whose `keys[0]` exactly matches `morality.paths[N].loreRef`. The path-pool dropdown should preserve `morality.virtues[N].options[]` two-state toggling for Conscience/Conviction and Self-Control/Instinct (active option conventionally driven by selected path — see the new lookup entries for canonical Virtue pairings, including Path of Night's "Cold variant").

### Added — Phase 5 step 5.5: 3-way density toggle (compact / cozy / roomy) (2026-05-10)

User-facing density preference shipped end-to-end: a 3-way pill toggle in the actions row, with `state.sheet.density` per-character persistence and `data-density` attribute branching on the sheet root. Default `cozy` matches the JSX prototype.

- **`extension/RPG-Extension-RP-Mode.css` `:root`** — appended 5 density variables (`--mrrp-density-pad-x`, `-pad-y`, `-gap`, `-row-h`, `-fs`) with cozy defaults, after the existing Phase 4 token block. No existing token modified.
- **`extension/RPG-Extension-RP-Mode.css` end-of-file** — new section `/* ── Phase 5 step 5.5: density toggle ── */` with three `.mrrp-sheet[data-density="compact|cozy|roomy"]` selector blocks branching the four density values, plus `.mrrp-density-toggle` pill-group classes (`__label`, `__btn`, `__btn[aria-pressed="true"]`, `:hover`, `:focus-visible`).
- **`extension/RPG-Extension-RP-Mode.js`** — added `density: "cozy"` to `blankSheet` defaults (~line 1389); set `data-density` attribute on `state.mountEl` immediately after creation in `mrrpP3RenderSheet` (~line 3702); appended density toggle UI in the actions-row block (~line 3756) iterating `["compact","cozy","roomy"]` to render three buttons with `aria-pressed` selection. Click handler writes `state.sheet.density`, sets `data-density`, calls `saveSheet(state.chatId, state.sheet)`, then `renderSheet()` — same save-then-re-render pattern as other actions-row controls.
- **Mirror discipline**: same restructure in `Marinara-RPG-Extension` with `mrr-` namespace (sheet at line 3509, actions row at line 3574, blankSheet at line 1086).
- **Embedded CSS regenerated** — `node tools/embed-css.mjs` re-ran in both repos as the orchestrated final step after Phase 5 step 5.4 also landed; both `EMBEDDED_CSS` strings contain Phase 5 step 5.5 + Phase 5 step 5.4 markers verbatim.
- **Follow-up — density-aware components not yet wired.** This step ships the toggle infrastructure and root-level variable swap. Existing components still use literal `--mrr(p)-pad`/`--mrr(p)-gap`/`13px` references — they won't visually respond to the toggle until a separate cycle threads `var(--mrr(p)-density-*)` through them. Recommend that follow-up land as its own slice.

### Added — Phase 5 step 5.4: derived tooltip math via `derivedStats[].tooltipFormula` (2026-05-10)

Hover any derived stat that declares a `tooltipFormula` (Plan B step 1 schema add) and see the breakdown: `Soak (Bashing): 7 = 4 + 3` with each contributor's source on its own line. Closes the gap where existing valueFormula tokens of the form `{bonuses:Bashing Soak}` silently evaluated to 0.

- **`extension/RPG-Extension-RP-Mode.js`** — added `mrrpSubstituteTokens(formula, ctx)` and `mrrpComputeTooltipBreakdown(derived, ctx)` helpers (lines 5583-5654). Token vocabulary: `{StatName}` (flat ctx lookup), `{StatName_mod}` (D&D-style modifier), `{bonuses:Key}` (NEW — resolves `equippedBonuses(Key).value`, summed across equipped items), `{Level}` and `{tierBonus}` (forward-compatible — currently 0 in derived contexts).
- **`extension/RPG-Extension-RP-Mode.js`** — extended `renderValue` (lines 5656-5720) inside `mrrpP3RenderDerivedSection`: when `tooltipFormula` is present, computes the substituted breakdown and emits it as `title=` on the value cell after `calc.textContent` is set in `refreshAutocalc`. Coexistence: when `valueFormula` is also present, `valueFormula` continues to drive the displayed number; `tooltipFormula` only drives the tooltip text. When only `tooltipFormula` is present, it computes BOTH the displayed value AND the tooltip.
- **`extension/RPG-Extension-RP-Mode.css` end-of-file** — new section `/* ── Phase 5 step 5.4: derived tooltip math ── */` (lines 2441-2459) with `.mrrp-row__value--autocalc[title]:not([title=""])` selectors: `cursor: help`, dotted underline via `--mrrp-text-faint`, hover state shifts underline to `--mrrp-accent-line`.
- **Mirror discipline**: same in `Marinara-RPG-Extension` with `mrr-` namespace (helpers at line 5454, renderValue edits at line 5527, CSS at line 2456).
- **Active formulas verified to render**: Exalted Soak (Bashing) `{Stamina} + {bonuses:Bashing Soak}`, Soak (Lethal) `{Stamina} + {bonuses:Lethal Soak}`; D&D Armor Class `10 + {Dexterity_mod} + {bonuses:Armor Class}`, Initiative `{Dexterity_mod} + {bonuses:Initiative}`. All compute correctly via in-session smoke test.
- **BEHAVIOR CHANGE — D&D Armor Class** is now autocalc-driven (no manual stepper) because its `tooltipFormula` is present without a `valueFormula`. Players who previously typed AC numbers directly will now need an inventory item with `bonuses: [{target: "Armor Class", value: N}]` equipped. This matches the long-term Soak-style architecture but is a UX shift worth noting. If unwanted, gate the promotion behind a `derived.computeFromTooltipFormula: true` opt-in flag.

### Added — Plan B step B.2: Exalted content cleanup (2026-05-10)

Five Exalted content updates following the new schema fields from Plan B step 1.

- **`rulesets/exalted3e/ruleset.json`** —
  - Removed `Sorcerous Motes` from `derivedStats[]` (now lives authoritatively in `resources[]` per Plan B step 1).
  - Added `"derived:Sorcery Circle"` and `"derived:Sorcerous Motes"` to `sections.hidden[]` (the latter as belt-and-suspenders against any cached state still referencing the deleted entry).
  - Added new state category `Initiative` with values `Active` and `Crashed` to model the canonical Exalted 3e initiative-crashed condition.
  - Prepended `Suppressed` option to the anima banner state values (covers no-motes-spent / Charm-damped / non-Exalt cases).
- **`rulesets/exalted3e/lorebook.json`** — no changes; the pre-existing `ex3-rules-sorcery-circles` entry (`Rule: Sorcery — The Three Circles`, covering Terrestrial / Celestial / Solar progression) already satisfies the spec's intent. The duplicate-id collision was caught and rolled back during implementation.
- **Mirror discipline**: byte-identical changes in `Marinara-RPG-Extension`.
- **Validation gates**: `node tools/validate-ruleset.mjs --all` PASS for exalted3e (v1.1.0) in both repos; mirror `diff` empty for ruleset.json.
- **Hand-off — Anima Banner now has 6 values** (Suppressed / Dim / Glowing / Burning / Bonfire / Iconic) — up from 5. Phase 5 step 5.6 visual treatment will need to accommodate the extra option (likely flex-wrap rather than fixed 5-column grid).
- **Out of scope this cycle (queued for follow-ups)**: Plan B step B.3 V20 morality detail fills, Plan B step B.4 fate-core / pathfinder2e content, ruleset version bump 1.1.0 → 1.2.0.

### Added — Phase 5 step 5.1: identity card visual restructure (2026-05-10)

Replaces the legacy `mrrp-sheet__identity-row` with a prototype-shaped compact identity card at the bottom of the sheet header. Pure-visual port from `~/projects/claude-design-updates/sheet.jsx` §3.4 — no functional, schema, or state-shape changes.

- **`extension/RPG-Extension-RP-Mode.css`** — added `.mrrp-identity` parent + 7 child classes (`__avatar`, `__main`, `__name`, `__sub`, `__sub-item`, `__sub-label`, `__sub-input`, plus `:focus` states). Type scale: 16px/600 borderless name input (`var(--mrrp-text)`), 9px uppercase letter-spacing 0.1em sub-labels (`var(--mrrp-text-faint)`), 12px borderless sub-inputs (`var(--mrrp-text-dim)`). Avatar slot uses 10px-mono dashed border placeholder text "PORTRAIT" matching prototype.
- **`extension/RPG-Extension-RP-Mode.js`** — restructured the tail of `renderSheetHeader` to emit the new card markup wrapping the existing identity inputs; added `mrrpRenderIdentitySubField` helper. Renderer branches on `Array.isArray(state.ruleset.identityFields)`: when present, iterates the schema-declared list; when absent, falls back to the legacy `header.raceLabel`/`classLabel` two-field pattern so all current rulesets (Exalted Type/Caste, D&D Class/Race, V20 Clan/Sect) keep working unchanged.
- **Save/load semantics preserved** — sub-inputs retain the 250ms debounced `saveSheet()` pattern + immediate-save-on-blur + click-stopPropagation. Name input edits `activeChar.name` with the same debounced pattern, calling `saveCharacters() + renderSheet()` so the character switcher dropdown updates live.
- **Embedded CSS regenerated** — `node tools/embed-css.mjs` re-ran; the `EMBEDDED_CSS` string in `extension/RPG-Extension-RP-Mode.js` is in sync.
- **Mirror discipline**: same restructure landed same-day in `Marinara-RPG-Extension` with the `mrr-` namespace.
- **Forward-compatible with Plan B's `identityFields[]` schema** — when rulesets begin declaring `identityFields[]` (Plan B step 1 also landed today), this renderer iterates them automatically.
- **Dead-code note (out of scope this cycle)**: the legacy `renderIdentityField` function and `mrrp-sheet__identity-*` CSS rules are now unused; left in place as harmless dead code, prune in a follow-up commit.

### Added — Plan B step 1: ruleset schema additions for resources, derived tooltips, ability grouping, and morality (2026-05-10)

Backwards-compatible JSON-only schema additions to `schema/ruleset.schema.json` plus per-ruleset population. Unblocks Phase 5 steps 5.3 (Resources cluster), 5.4 (Derived tooltip math), and 5.6 (per-ruleset visual tweaks).

- **`schema/ruleset.schema.json`** — added five top-level structural areas:
  - **`sections`** with `order[]` (enum-typed string[] declaring section render order) and `hidden[]` (string[] hiding sections or `derived:<name>` individual derived stats).
  - **`resources[]`** — array of Resource objects: `{id, label, type: bar|dice|counter|pool|custom, max?, current?, die?, group?, color?, quickButtons?, rendererConfig?}`. State tag format: `[mrrp-state: resource:<id>=N]`.
  - **`derivedStats[].tooltipFormula`** — token-substituted formula string, drives the Phase 5 step 5.4 tooltip math. Coexists with the existing `formula` (GM prose hint) and `valueFormula` (autocalc) fields. Note: Plan B doc literally specified `derived[].formula` but `derivedStats[].formula` already exists with different semantics — `tooltipFormula` is the minimum-disruption naming that delivers Plan B's intent without a breaking rename.
  - **`abilities.groups[]`** — array of `{id, label, members[]}` for V20 Talents/Skills/Knowledges grouping; optional and ignored by rulesets that don't group abilities.
  - **`morality`** — section type with `rating {label, min, max, default}`, `paths[]`, and `virtues[]` (each virtue supports `options[]` for paired-choice virtues like Conscience/Conviction).
- **`rulesets/exalted3e/ruleset.json`** — populated `sections.order[]` (12 sections), `resources[]` (HP, Motes Personal, Motes Peripheral, Willpower, Sorcerous Motes), `derivedStats[].tooltipFormula` on Soak (Bashing) `{Stamina} + {bonuses:Bashing Soak}` and Soak (Lethal).
- **`rulesets/vtmv20/ruleset.json`** — populated `sections.order[]` (13 sections including new `morality` and `disciplines` slots), `sections.hidden[]` (12 entries hiding each Discipline from Derived per Phase 5 step 5.6 spec), `resources[]` (Blood Pool / Willpower / Health-as-`custom` with `rendererConfig.component: "v20-health-track"`), `abilities.groups[]` (Talents/Skills/Knowledges with verbatim V20 member lists), `morality` section (Path Rating + 5 paths + 3 virtues with Conscience/Conviction and Self-Control/Instinct as paired-choice).
- **`rulesets/dnd5e/ruleset.json`** — populated `sections.order[]` (13 sections), `resources[]` (HP, Hit Dice, Spell Slots 1st-3rd grouped under "Spell Slots"), `derivedStats[].tooltipFormula` on Armor Class `10 + {Dexterity_mod} + {bonuses:Armor Class}` and Initiative `{Dexterity_mod} + {bonuses:Initiative}`.
- **`rulesets/fate-core/ruleset.json`** — no edits; new fields are optional and the file already validates against the new schema.
- **`rulesets/pathfinder2e/ruleset.json`** — file remains absent (pre-existing condition flagged for separate triage); not created by this work.
- **Validation gates**: `node tools/validate-ruleset.mjs --all` PASS for every existing ruleset.json in both repos; mirror diff between RP and GM ruleset.json files returns empty (byte-identical content; only intentional namespace differences in schema files preserved).
- **Out of scope this cycle (queued for follow-ups)**: removing the now-redundant Sorcerous Motes from Exalted `derivedStats[]` (Plan B step B.2 content cycle), the Sorcery Circle hiding + lorebook ingestion (Plan B step B.2), the Initiative-crashed Exalted state and anima "none" option (Plan B step B.2), V20 morality data fills beyond schema-conforming defaults (Plan B step B.3).

### Added — Phase 4 design-token migration: oklch palette + Geist font stack (2026-05-10)

Visual layer of the UI rebuild ported from the JSX prototype at `~/projects/claude-design-updates/`. No functional or structural change — this is the appearance-only counterpart to the structural Phase 3 cutover that landed earlier this week.

- **`extension/RPG-Extension-RP-Mode.css` `:root`** — every color token migrated from hand-picked rgba/hex to perceptually-uniform oklch values matching the prototype's purple/dark mood (background `oklch(0.21 0.025 285 / 0.96)`, accent derived from new tweakable `--mrrp-accent-h: 280` / `-c: 0.12` / `-l: 0.78`). Token NAMES preserved across the migration so every existing CSS rule resolves unchanged. New tokens added per prototype: `--mrrp-bg-app`, `--mrrp-bg-input`, `--mrrp-hairline`, `--mrrp-hairline-strong`, `--mrrp-accent-soft`, `--mrrp-accent-line`, `--mrrp-text-faint`, `--mrrp-sans`.
- **Geist font stack** — `--mrrp-sans` introduced (`"Geist", "Inter", system-ui, ...`); `--mrrp-mono` updated to lead with `"Geist Mono"`. Applied via `font-family: var(--mrrp-sans)` on the `.mrrp-sheet` root so all descendant text picks it up. Geist falls back gracefully through the stack if not installed locally — no WOFF2 bundling this cycle (deferred until requested).
- **Shadow** — replaced single 8px-offset drop with the prototype's two-layer 24px + 6px composite for richer panel elevation.
- **`tools/token-migration-map.json`** — full mapping between every old token value, the new oklch value, and the prototype source variable. Includes an `added` section listing the new tokens and a `deferred_to_followups` section calling out density toggle, motion tokens, and light/dark mode (each is its own future cycle, intentionally not mixed in).
- **Embedded CSS regenerated** — `node tools/embed-css.mjs` re-ran; the `EMBEDDED_CSS` string in `extension/RPG-Extension-RP-Mode.js` is in sync with the standalone `.css` file (grep proof: oklch literal + `Geist Mono` both present).
- **Validation gates passed**: `node --check extension/RPG-Extension-RP-Mode.js` (clean) · `npm run validate-bundles` (5/5 PASS) · cross-namespace grep for `\bmrr-` in `extension/ schema/ tools/` (zero hits — namespace integrity preserved).
- **Mirror discipline**: same migration landed same-day in `Marinara-RPG-Extension` (GM-mode) with the `--mrr-*` namespace.
- **Out of scope this cycle (queued for follow-ups)**: density toggle (step 4.4 — touches state schema), motion tokens (step 4.5 — prototype defines none to port), light-mode adaptation (prototype is single-mode dark too).
- **Visual verification gate**: re-import `RPG-Extension-RP-Mode.js` + `RPG-Extension-RP-Mode.css` from Marinara → Settings → Extensions, hard-refresh, then open an Exalted or D&D character sheet to confirm the deeper purple surface + Geist body type land correctly.

### Added — V:TM V20 ruleset bundle (`mrrp-vtmv20`) — first-pass draft from training + internet research (2026-05-09)

New ruleset bundle for **Vampire: The Masquerade 20th Anniversary Edition** (V20, 2011 Onyx Path). Phase 1 of a multi-phase build — drafted from model training plus multi-source web research (Onyx Path corebook references, Anarch State wiki for the canonical generation chart, Saligia / White Wolf wiki cross-checks for clan/Discipline/sect content, Onyx Path forums for V20-specific botch/Willpower/specialty rules, and the live Dark Pack Agreement text from worldofdarkness.com for IP boundaries). Phase 2 (V20 corebook ingestion to fill gaps) is held in reserve; see EVALUATION.md in the per-session work dir for the post-Phase-1 gap report and the corebook-ingestion verdict.

- **`rulesets/vtmv20/ruleset.json`** — full V20 character sheet template: 9 Attributes (Physical/Social/Mental), 30 Abilities (Talents/Skills/Knowledges, category in tooltip), 9 common + 3 clan-unique Discipline ratings as derived stats, 3 Virtues, Generation, Blood Pool bar, Willpower bar, Humanity bar, Path Rating, Initiative auto-calc, V20 7-level Health Track with B/L/A damage types (Aggravated soak = Fortitude only), Backgrounds & Merits, Morality Track selector, Frenzy State, Hunger Tier. Header labels: Clan / Sect.

- **`rulesets/vtmv20/lorebook.json`** — 52 keyword-triggered reference entries: mechanics rules (resolution, botch, Willpower, generation table, Beast/Frenzy, hunger/feeding, Blood Bond, torpor/Final Death, Six Traditions); 13 V20 clans + 4 V20 bloodlines (Caitiff, Daughters of Cacophony, Salubri, Samedi); 9 common Disciplines with all 5 named power levels each + 3 clan-unique Disciplines (Necromancy, Thaumaturgy, Vicissitude); 4 sects; Humanity hierarchy + 4 Path hierarchies of sins (Honorable Accord, Caine, Beast, Night); sub-agent doc. Every entry under the 2500-char lorebook budget.

- **`rulesets/vtmv20/gm-agent.md` + `agents.json`** — main Storyteller agent prompt framed for **roleplay mode**, working alongside Marinara's default world-state / prose-guardian / continuity / expression agents. Targets the main narration model (not a single GM model). Drops the 50-character reputation tag cap workaround that GM-mode requires (RP-mode lifts the cap). Otherwise V20-canonical mechanics: variable difficulty 6-9, botch on any 1 with zero successes, Willpower auto-success (one per turn cap), Potence auto-successes, Celerity end-of-turn extra physical actions, generation-keyed Blood Pool & per-turn spend caps, V20 health track with Aggravated soak rule, state-mutation tag schema (`[mrrp-state: ...]`), Discipline activation tag.

- **`rulesets/vtmv20/agents/*.md`** — five optional sub-agents installed DISABLED (toggle in Marinara → Settings → Agents): `state-mutator` (emit `[mrrp-state: ...]` tags inline), `state-reminder` (bullet PC state every turn), `combat-adjudicator` (V20 combat math; sleeps in social scenes), `lore-query` (answer rules questions from lorebook + RAW; sleeps otherwise), `npc-bookkeeper` (track Kindred NPCs across turns). All five framed for RP-mode (working with default narration model and engine sub-agents).

- **`rulesets/vtmv20/INSTALL.md`** — file-import instructions, Dark Pack compliance notes, sub-agent enablement guide, cross-OS install commands, companion-install pointer to the GM-Mode repo. Notes the lifted reputation cap and the RP-mode multi-agent collaboration model.

- **`rulesets/vtmv20/bundle.json`** — assembled single-file install (52 lorebook entries + 6 agents + full ruleset). Validates clean. `mrrp-bundle` envelope; `mrrp-vtmv20` ruleset id; idempotent re-install supported.

- **Validation gates passed**: `npm run validate-rulesets` (vtmv20 PASS) · `npm run validate-bundles` (vtmv20 PASS) · `node --check extension/RPG-Extension-RP-Mode.js` (clean) · namespace audit — zero standalone `mrr-` hits inside `rulesets/vtmv20/` (only `mrrp-` tokens, as required).

- **Dark Pack compliance**: license string in `ruleset.json` carries the Paradox copyright notice + "NOT official World of Darkness material" disclaimer per the [Dark Pack Agreement](https://www.worldofdarkness.com/dark-pack). All flavor text is original-prose paraphrase; mechanical references only; no contiguous V20 corebook quotation longer than ~30 words. Display the Dark Pack logo on the project README and any release page.

- **Companion bundle**: same-day landing in `Marinara-RPG-Extension` repo as `rulesets/vtmv20/` with the `mrr-vtmv20` namespace and GM-mode-framed Storyteller (includes 50-char cap workaround). Both bundles can coexist on a single Marinara install.

### Added — Phase 3.6 UI port: inventory (equipment) + abilities (charms/spells) migrated behind feature flag (2026-05-08)

Final body slice. With `state.sheet.useNewRenderer === true`, **all ten** sheet sections now render through Phase-3 wrappers. Item-editor dialog, Spellbook flyout, Intimacies flyout, and Items flyout panels remain classic — each is a separate flyout subsystem deferred to fine-tuning sessions.

- **`mrrpP3RenderInventorySection(parent)`** — Phase-3 section frame around the equipment list, using newly-extracted `renderInventoryList(parent)` helper. Helper lifted from classic `renderInventory` so the equipment list renders identically inside either a classic `.mrrp-section` parent (flag-OFF) or Phase-3 `.mrrp-p3-section__body` parent (flag-ON). Avoids double-section-header. All inventory functionality preserved.

- **`renderInventoryList(parent)`** — refactored from classic `renderInventory`'s inline body. DOM byte-identical to before.

- **`mrrpP3RenderAbilitiesSection(parent)`** — same pattern as Intimacies wrapper: Phase-3 section frame + button → `showSpellbook(true)`. Section omitted when ruleset declares no abilities.

- **Section dispatch swap** — two lines in `mrrpP3RenderSheet`.

- **Engine + classic helpers untouched** — except classic `renderInventory` body refactored to delegate (DOM identical).

- **No new CSS this slice.**

- **All 10 sections now Phase-3 framed under flag.**

### Added — Phase 3.5 UI port: skills + states + conditions + backgrounds + intimacies migrated behind feature flag (2026-05-08)

Marathon slice. With `state.sheet.useNewRenderer === true`, the Phase-3 sheet now renders **eight** sections via Phase-3 wrappers — Attributes (3.3), Derived (3.4), Saves (3.4), Skills, States, Conditions, Backgrounds, and Intimacies. Only Inventory (item-editor flyout) and Abilities/Charms (charm tree) remain on classic helpers.

- **`mrrpP3RenderSkillsSection(parent)`** — iterates `state.ruleset.skills` and calls `mrrpP3RenderSkillRow` per skill with the full opt set. Autocalc-vs-manual gate on `state.ruleset.resolution.skillBonusFormula`. Tier-cycle persists via `state.sheet.skillProficiency[skill.name]`. Specialties round-trip through `state.sheet.skillSpecialties[skill.name]`. Section title resolves to "ABILITIES" for Exalted, "SKILLS" elsewhere.

- **`mrrpP3RenderCustomSkillRow(parent, sk, idx)`** — inline Phase-3 row for custom skills with debounced editable name + linked-attribute select + value input + remove. Reuses classic `addCustomSkill` / `removeCustomSkill`.

- **`mrrpP3RenderStatesSection(parent)`** — name + select per state; persists to `state.sheet.states[name]`. Inline Phase-3 row.

- **`mrrpP3RenderConditionsSection(parent)`** — active list with effect chips (disadv/adv from ruleset definitions) + × remove. Add-condition select with "(other — type a name)" custom option. Reuses classic `addCondition` / `removeCondition`.

- **`mrrpP3RenderBackgroundsSection(parent)`** — debounced editable name + value input (clamped to `cfg.min/max`) + × remove. Respects `cfg.textOnly` (D&D feats). "+ Add Background/Feat" footer button.

- **`mrrpP3RenderIntimaciesSection(parent)`** — Phase-3 section frame with single "Intimacies (N)" button that opens the existing classic flyout. Full flyout migration deferred to Phase 3.6+.

- **Section dispatch swap** — five lines in `mrrpP3RenderSheet`. Classic `renderSheet` body untouched.

- **Engine + classic helpers untouched** — `statContext`, `equippedBonuses`, `tierForSkill`, `resolveTierBonus` and the seven classic renderers all unchanged.

- **No new CSS this slice** — Phase-3 wrappers reuse Phase 3.1 row primitives + classic row chrome inside Phase-3 section bodies. CSS file unchanged.

- **v1 retreats** — per-specialty value editing not migrated (primitive shows static dice); custom skill name uses inline row not primitive; Intimacies flyout still classic.

### Added — Phase 3.4 UI port: derived (bars + damage track) + saves migrated behind feature flag (2026-05-08)

Third slice of the multi-session renderSheet rewrite. With `state.sheet.useNewRenderer === true`, the Phase-3 sheet now renders three full sections via the Phase 3.1 primitives — Attributes (shipped in 3.3), Derived (bars + damage track, this slice), and Saves (this slice). Every other section still routes through classic helpers; the flag-OFF default is unchanged.

- **`mrrpP3RenderDerivedSection(parent)`** — iterates `state.ruleset.derivedStats` and dispatches per `renderAs`. Bar entries call `mrrpP3RenderBar` via the new `mrrpP3RenderDerivedBar` bridge; track entries call `mrrpP3RenderDamageTrack` via the new `mrrpP3RenderDerivedTrack` bridge; non-bar/non-track entries fall through to classic `renderValue` (parent-agnostic — safe to call from any container). State contracts preserved verbatim: `state.sheet.derived[name]`, `state.sheet.derivedMax[name]`, typed-damage `state.sheet.track[name]`, `state.sheet.extraTrack[name]`. Mutations trigger full `renderSheet` re-render — Phase-3 path resets `barRefreshers` AND `derivedBonusRefreshers` at its head so refresher cleanup is automatic.

- **`mrrpP3RenderDerivedBar(parent, d)` + `mrrpP3ComputeBarMax(d)`** — bar-state bridge. Max precedence mirrors classic: `derivedMax[name]` override > `maxFormula` evaluated on `statContext()` > literal `max` > `Math.max(DEFAULT_BAR_MAX, current)` for fresh sheets. v1 retreats: bar bonus pill (`.mrrp-row__bonus`) and roll button for derived stats with `rollFormula` are NOT migrated — primitive doesn't expose them; revisit in a future slice.

- **`mrrpP3RenderDerivedTrack(parent, d)`** — typed-damage bridge. Severity-descending fill: types sorted A,L,B (high→low) so the leftmost cells carry the worst damage. Cell click semantic: filled cell heals that type, empty cell takes the lightest type — matches classic `renderTrack` single-click fallback. `onAddBox` extends `state.sheet.extraTrack[name]` with `{label, penalty}`; `onRemoveBox` pops the last entry; `onHeal("worst")` drops one of the highest-severity damage in the typed counter, `onHeal("all")` zeros every type. v1 retreat: classic Exalted-style Take-B/L/A buttons NOT migrated — primitive doesn't expose them, this is a JSX-prototype UX choice not a regression.

- **`mrrpP3RenderSavesSection(parent)`** — iterates `state.ruleset.saves` and builds Phase-3 SaveRows via `mrrpP3RenderSaveRow`. Tier state shared with skill-proficiency map (`state.sheet.skillProficiency[save.name]` — same key `cycleTier` mutates); wrapper writes the same key on tier-cycle. Total bonus computed via the same math classic `refreshSaveBonus` uses (`skillBonusFormula` substitution OR `attrMod + tierBonus` fallback). Roll callback delegates to existing `quickRollForSave(save)`; primitive's `onRoll` args ignored since `quickRollForSave` self-computes from the save object.

- **Section dispatch swap** — in `mrrpP3RenderSheet`, the "saves" branch becomes `mrrpP3RenderSavesSection`; the "derived" branch becomes `mrrpP3RenderDerivedSection`. Two-line edit. Classic `renderSheet` body untouched — flag-OFF still routes through `renderSaves` / `renderDerived`.

- **Engine functions untouched** — `statContext`, `equippedBonuses`, `tierForSkill`, `resolveTierBonus` unchanged. Classic `renderDerived` / `renderBar` / `renderTrack` / `renderSaves` function bodies unchanged.

- **No CSS additions this slice** — primitives shipped in 3.1 already include `.mrrp-p3-bar`, `.mrrp-p3-bar--damage`, `.mrrp-p3-track`, `.mrrp-p3-cell--{B,L,A}`, `.mrrp-p3-track-tools`, `.mrrp-p3-row--save`, `.mrrp-p3-save__bonus`. CSS file unchanged.

### Added — Phase 3.2 + 3.3 UI port: panel-frame factory + renderer cutover behind feature flag (2026-05-08)

Second slice of the multi-session renderSheet rewrite. Panel-frame chrome translated from `~/projects/claude-design-updates/panel-frame.jsx`, and a parallel `mrrpP3RenderSheet` lands behind a `state.sheet.useNewRenderer` flag so users can opt into the new path while the classic renderer remains the default. Initial cutover migrates only the Attributes section to the Phase 3.1 primitives; every other section still routes through the classic helpers.

- **`mrrpP3CreatePanel(parent, opts)`** — vanilla-JS port of the JSX prototype's `DraggablePanel` (`panel-frame.jsx`). Floating panel with drag-from-head + 8 resize handles (4 edges + 4 corners) + `localStorage` position/size persistence + viewport re-clamp on `window.resize`. Returns `{ panel, head, body, dispose }` so callers populate the body and clean up event listeners on close. Drag from head ignores button / input / select / textarea targets via `e.target.closest`. The SE corner gets a small SVG glyph for affordance. Currently unused by the running renderer — held in reserve for future Session 3.4+ flyouts.

- **`mrrpP3RenderSheet()` parallel renderer** — alternative entry point that REUSES the existing `.mrrp-sheet` shell + `makeDraggable` / `makeResizable` so toggling renderers preserves the user's saved sheet position. Section dispatch identical to classic except the Attributes section, which routes to the new primitives via `mrrpP3RenderAttributesSection`. Every other section (Skills, Saves, Derived, States, Conditions, Intimacies, Backgrounds, Inventory, Abilities) still calls the classic `renderXxx` helpers.

- **`mrrpP3RenderAttributesSection(parent)`** — wires the Phase 3.1 `mrrpP3RenderSection` + `mrrpP3RenderAttrRow` primitives to `state.sheet.attributes` + `saveSheet`. Modifier slot populates from `statContext()[<attr>_mod]` for D&D-family rulesets that declare `modifierFormula`; pool systems pass `undefined` and the modifier slot collapses.

- **Feature flag — `state.sheet.useNewRenderer`** — defaults to `false` in `blankSheet`. The classic `renderSheet()` entry now dispatches to `mrrpP3RenderSheet()` when the flag is `true`. The flag persists across sessions: `mergeSheet` adds `if (override.useNewRenderer === true) base.useNewRenderer = true` so the user's preference survives reload.

- **UI toggles** — the classic actions row gains a `🧪 Try Phase 3 renderer` button. The new actions row gains a `↩ Use classic renderer` button. Both states reachable from the UI.

- **Section open/closed persistence** — `mergeSheet` preserves `state.sheet.sectionCollapse`.

- **Panel-frame CSS (`mrrp-p3-panel*` namespace)** — ~110 lines covering panel root, head/title/title-meta/close, body slot, and 8 resize handles. Re-embedded via `tools/embed-css.mjs`.

- **Engine functions still untouched.** Sacred this phase too.

- **Deferred** (continuing the multi-session port): `panel-frame` factory has no caller yet — foundation for Session 3.4+ flyouts. Sections beyond Attributes migrate one at a time across Sessions 3.4–3.N. Token migration still deferred.

### Added — Phase 3.1 UI port: row primitives extracted from sheet.jsx (2026-05-08)

First slice of the multi-session renderSheet rewrite. Seven foundational row primitives translated from the Claude-Design prototype's `~/projects/claude-design-updates/sheet.jsx` into vanilla-JS DOM-builder functions, sitting alongside the existing 7K-line `renderSheet` so a future-session cutover can compose them mechanically. No behavioral change this commit — primitives are uncalled by the running renderer.

- **`mrrpP3RenderSection(parent, opts, bodyFn)`** — collapsible card matching `sheet.jsx:17`. Open/closed state persists in `state.sheet.sectionCollapse[id]` when `id` is provided. Click on header toggles + saves + re-renders; clicks on actions / right slot don't propagate.
- **`mrrpP3RenderStepper(parent, opts)`** — −/+ stateless buttons matching `sheet.jsx:34`. Clamps via `mrrpP3Clamp` and calls `onChange`.
- **`mrrpP3RenderAttrRow(parent, opts)`** — attribute row (5-col grid) matching `sheet.jsx:44`. Composes `mrrpP3RenderStepper`. Modifier slot collapses when `opts.modifier` is undefined/null.
- **`mrrpP3RenderSkillRow(parent, opts)`** — skill row (4-col grid) matching `sheet.jsx:60`. Largest primitive — handles autoCalc total breakdown tooltip, clickable tier pill that cycles `opts.tiers`, gear bonus pill, in-place specialty chips with delete + click-to-roll, an inline specialty editor toggleable via `+ spec` button, and an optional `×` delete button.
- **`mrrpP3RenderSaveRow(parent, opts)`** — save row (4-col grid) matching `sheet.jsx:132`. Always autoCalc; total bonus computed by caller; tier pill cycles like SkillRow's.
- **`mrrpP3RenderBar(parent, opts)`** — resource bar matching `sheet.jsx:149`. Auto-color: ratio < 0.3 → bad, < 0.65 → warn, else ok (override via `opts.fillVariant`). Quick-button array supports `{label, delta}` deltas applied to current.
- **`mrrpP3RenderDamageTrack(parent, opts)`** — Exalted damage track matching `sheet.jsx:180`. Counts B/L/A from `track.filled`; renders cell buttons whose click fires `onCellClick(i)`; track tools support add -0/-1/-2 boxes, remove last, Heal worst, Heal all (disabled when no damage).

CSS additions (~500 lines) under the **`mrrp-p3-*` namespace** prevent collision with the running renderer's existing `.mrrp-section` / `.mrrp-row` / `.mrrp-stepper` / `.mrrp-bar` rules. Both `extension/RPG-Extension-RP-Mode.css` and the embedded `EMBEDDED_CSS` string are in sync via `tools/embed-css.mjs`. Density values (12/8/30/13 cozy defaults from UI-build.md §2.3) are inlined since the extension doesn't yet ship `--density-*` tokens; the token-migration phase will swap them.

Function names use the `mrrpP3*` prefix for the same reason — `renderAttrRow` / `renderSkillRow` / `renderSaveRow` / `renderBar` already exist in the extension at the running-renderer's call sites; same-name redeclarations would shadow and break the live UI.

The new primitives have no caller this session by design — they're foundation for Session 3.2 (panel-frame port) and Session 3.3+ (renderSheet body cutover). Engine functions (`statContext`, `equippedBonuses`, `tierForSkill`, `resolveTierBonus`) are untouched. State-mutator parser (Phase 2) is untouched. Bundle install validators all PASS.

### Added — Phase 2 UI port: state-mutator write-back + mote-pool plumbing (2026-05-08)

Phase 2 of the multi-session UI port. Closes the read/write asymmetry Phase 1 left open: agents could SEE the new XP and commitment state but couldn't MODIFY it. Phase 2 wires the write path on three surfaces — agent tags, inventory dialog, and item delete — so every mutation route honors the same caps and pool budget.

- **State-mutator parser — five new field branches.** `applyStateMutation` in `extension/RPG-Extension-RP-Mode.js` gains branches for:
  - `[mrrp-state: field="xp" delta="±N"]` and absolute `current/level/next/total` (pool-style rulesets bump both `current` and `total` on positive delta; mixed delta+absolute rejected as ambiguous)
  - `[mrrp-state: field="attunement" item="Name" attuned="true|false"]` (cap-3 enforced at write time; exclusivity check against invested + mote commitment)
  - `[mrrp-state: field="investiture" item="Name" invested="true|false"]` (cap-10; mirror exclusivity)
  - `[mrrp-state: field="commitment" item="Name" motes=N pool="Personal|Peripheral"]` (atomic pool change, negative-floor refusal, exclusivity vs attuned/invested)
  All branches surface error toasts via `warn()` on rejection and route success through `finalizeMutation` for the existing saveSheet → renderSheet → mutationLog → toast pipeline.

- **Cache fix on snapshot read sites.** `buildSyncFields` and `buildSheetForPrompt` no longer short-circuit on the `state.sheet.attunedCount` / `investedCount` caches. All four read sites recompute from inventory each call so an agent-side write that bypasses `openItemDialog` (the Phase 2.B parser) still produces correct counts in the next snapshot fire. Cache fields stay on disk for legacy consumers but are no longer trusted.

- **Delete-time mote restore.** `deleteItem` now restores committed motes to `state.sheet.derived[motePool]` before persisting. Removing an Exalted item with `moteCommitment > 0` previously leaked those motes — the field went to GC but the pool subtraction stayed forever. Phase 2 closes that.

- **Mote-pool live decrement on item save.** The `openItemDialog` save handler (mote model) now propagates `moteCommitment` + `motePool` deltas to `state.sheet.derived[motePool]` atomically. Pool change (Personal → Peripheral) restores the old commit to the old pool first, then debits the new commit from the new pool. Negative-floor check refuses commits that would deplete a pool below 0 — the inline message slot surfaces the error and the dialog keeps the prior commitment.

- **Agent docs — `gm-agent.md` (D&D 5e + Exalted 3e).** New `## State-mutator tags` sections teaching the narration model when to emit `xp`, `attunement`, and `commitment` tags. D&D's section covers the cap-3 attunement rule and the formula-style XP behavior; Exalted's section covers the pool-style XP accounting (delta bumps `current` AND `total`) and the two-pool mote commitment model.

- **Lorebook entries — D&D 5e + Exalted 3e + PF2e.** Keyword-triggered context injections so the agent can answer "how does attunement work?" / "what's mote commitment?" / "how does PF2e investiture work?" using SRD-aligned content. Each entry includes the corresponding state-mutator tag form so the agent doesn't have to consult `gm-agent.md` to remember the syntax. PF2e gets two entries (investiture + state-mutator XP tags) because PF2e's bundle.json embeds the lorebook directly without a sibling `gm-agent.md`.

- **Three pool writers, one pattern.** All three derived[motePool]-mutating paths — `deleteItem`, `openItemDialog` save handler, and the new `applyStateMutation` commitment branch — follow the same restore-old + debit-new pattern with negative-floor refusal. Side-effect ordering is consistent so a future cross-path interaction (delete-an-item-with-motes immediately after a state-mutator commit) does the right thing.

- **Engine functions still untouched.** `statContext`, `equippedBonuses`, `tierForSkill`, `resolveTierBonus` — sacred this phase too.

- **Deferred** (continuing the multi-session port, tracked in `MEMORY/WORK/20260508-marinara-ui-port/ISA.md`): the bigger work — design-token migration to oklch + Geist fonts, full `renderSheet` rewrite from `panel-frame.jsx`, item-bonus schema upgrade `{target,value,kind}` → `{stat,delta}`. Decision point for next session: tokens first (CSS-only, shallower) or renderer first (JS, deeper but unblocks the rest).

### Added — Phase 1 UI port: commitmentModel + XP card + snapshot extension (2026-05-08)

First Phase of the multi-session UI port that merges the Claude-Design prototype at `~/projects/claude-design-updates/` into both extensions. Phase 1 ships the data plumbing, the visible XP card, and the agent-facing snapshot. Subsequent phases will translate the rest of the React/JSX prototype (panel-frame, dialog system, full token migration) into the existing vanilla-JS form.

- **`commitmentModel` declaration in `ruleset.schema.json`.** New optional enum property on the ruleset root: `"attuned"` (D&D 5e), `"invested"` (Pathfinder 2e), `"mote"` (Exalted 3e), or `null` (default — ruleset has no commitment mechanic). Existing rulesets are unaffected; the field is optional with a null default.
- **`xpTable` declaration in `ruleset.schema.json`.** New optional array property on the ruleset root: ordered `{level, xp}` entries that drive the XP card's next-level threshold + progress bar. Pool-based rulesets omit this; their XP card shows current/total + a +1 button.
- **Reference rulesets declare commitmentModel.** `dnd5e/ruleset.json` declares `"attuned"` + 20-entry SRD `xpTable`. `exalted3e/ruleset.json` declares `"mote"` (no xpTable; pool system). `pathfinder2e/bundle.json` declares `"invested"` + 20-entry 1000/level `xpTable`.
- **Item commitment fields in inventory editor.** New "Magic / Commitment" section between Hardness/Overwhelming and Bonuses, gated by `ruleset.commitmentModel` and the equipment category. Boolean models (attuned / invested) render a checkbox with cap enforcement (3 / 10) and an inline error in the existing dialog message slot when the cap is hit. The mote model renders an integer commit input plus a Personal/Peripheral pool select. Items now persist `attuned`, `invested`, `moteCommitment`, `motePool` fields; `normalizeInventoryItem` heals legacy saves into the new shape.
- **Cap enforcement counters on the sheet.** New `state.sheet.attunedCount` and `state.sheet.investedCount` derived caches refreshed on every inventory write so the editor's cap check stays cheap and the snapshot always shows the current budget.
- **XP card slotted between identity and section list.** New `renderXpCard` function called from `renderSheet` between `renderSheetHeader` and the `sectionsforEach` loop. Two layouts driven by `ruleset.resolution.mode`: single-roll (D&D, PF2e) shows level + current/next + 4px progress bar fed by `ruleset.xpTable`; dice-pool (Exalted) shows current + total earned + a "+1 XP" button (a pure int accumulator that increments BOTH fields atomically). Skipped silently for any other resolution mode (Fate Core uses Fate Points, not XP). Live updates: editing level or current refreshes the next-display and bar fill in place — no full sheet rebuild, focus preserved. Lazy field healing: tolerates a saved sheet without xp fields and seeds defaults in place, so legacy sheets work without a `mergeSheet` migration.
- **Snapshot extension — `buildSyncFields`.** Writes new fields to `chats.customTrackerFields` so any agent reading the standard tracker surface sees XP and commitment state. Single-roll → `XP Level`, `XP Current`, `XP Next`. Dice-pool → `XP Current`, `XP Total Earned`. Boolean commitment → `Attuned` / `Invested` field formatted as `in-use / cap`. Mote model → `Mote Commitment` field with total + per-pool breakdown.
- **Snapshot extension — `buildSheetForPrompt`.** Adds an "Experience:" section before the Inventory block, annotates each inventory row with its commitment state (ATTUNED / INVESTED / committed:N motes), and adds a "Magic / Commitment:" summary block after the inventory list. The GM agent now sees XP progression and item commitment in every prompt fire, without the player having to re-explain it. Foundation for the agent-side rule-awareness work targeted before the 5/14 ship.
- **CSS additions for the XP card.** New `.mrrp-xp-card-*` classes in `RPG-Extension-RP-Mode.css` (134 lines) re-embedded into the JS string via `tools/embed-css.mjs`. Reuses existing `--mrrp-bg-elev`, `--mrrp-accent`, `--mrrp-mono`, etc. tokens — Phase 1 is additive, no token migration yet.
- **Engine functions intentionally untouched.** `statContext`, `equippedBonuses`, `tierForSkill`, `resolveTierBonus`, the dice payload schema, the chat-feed payload shapes — all preserved exactly. Autocalc paths sacred this run.
- **Deferred to subsequent sessions** (tracked in `MEMORY/WORK/20260508-marinara-ui-port/ISA.md`): full `renderSheet` rewrite, panel-frame port, dialog system port, design-token migration to oklch + Geist fonts, item-bonus schema upgrade `{target,value,kind}` → `{stat,delta}`, mote-pool live decrement on commit, `mergeSheet` defaulting xp + counters for legacy saved sheets.

### Added — Exalted: Hardness, Overwhelming, Intimacies, GM-mutable backgrounds + intimacies (2026-05-06)

- **Hardness on inventory items.** New non-negative integer field `hardness` on every inventory item. Defaults to 0 (chip hidden). Surfaces as a `Hardness N` chip on the inventory card and a "Hardness" numeric input in the equipment-edit dialog. Heals on legacy saves via `normalizeInventoryItem`.
- **Overwhelming on inventory items.** Companion field `overwhelming` (also non-negative integer). Surfaces as `Overwhelming N` chip and "Overwhelming" dialog input. Both fields are equipment-only in the dialog — non-equipment items hide the inputs and clear the values to 0 on save.
- **Intimacies sheet section + flyout panel (Exalted).** New `renderIntimaciesSection` mirrors the Charms/Spellbook flyout pattern. Floating panel groups entries into Minor / Major / Defining collapsible sections (state in `state.sheet.intimacyCollapse[degree]`, default closed). Each row: kind chip (Tie ⇄ Principle, click to toggle), debounce-saved text input, degree dropdown, optional target field (Tie only), Delete button. Per-degree `+ Add <Degree>` buttons plus a top-level `+ Add Intimacy`. State on sheet: `intimacies: []`, `intimacyCollapse: {}`. Healed on legacy saves via `mergeSheet`.
- **State-mutator: backgrounds.** GM agent can now adjust the on-sheet Backgrounds & Merits via `[mrrp-state: field="backgrounds" ...]` tags: `add="Name" rating="N"`, `remove="Name"`, or `name="Name" delta="±N"` (clamped to ruleset min/max). Symmetric with the existing conditions/inventory branches; goes through `finalizeMutation`/`showMutationToast` and respects `processedMessageIds` idempotency.
- **State-mutator: intimacies.** Companion branch: `[mrrp-state: field="intimacies" add="text" degree="..." kind="..." target="..."]`, `remove="text"`, or `text="text" degree="..."` / `kind="..."` to update an existing entry in place. Defaults: `kind="tie"`, `degree="minor"`. Removing a kind toggle off `tie` clears the target.
- **Lorebook entries (Exalted).** Three new keyword-triggered entries explain Hardness, Overwhelming damage, and Intimacies (Ties + Principles + Minor/Major/Defining degrees) in canon terms.
- **`gm-agent.md` (Exalted).** New sections teach the agent the four state-mutator tag combinations and remind it to honor Hardness vs Overwhelming when narrating combat outcomes.

### Added — Step 1 of character-library storage evolution: chat-independent sheet keys (2026-05-06)

- **Sheet data decoupled from `chatId`.** New `characterKey(characterId)` returns `mrrp-character-{characterId}` — the post-v0.2.1 home for sheet data. `loadSheet` tries this key first, falls back to the legacy `mrrp-sheet-{chatId}-{characterId}` once with auto-migration (copies forward, leaves legacy in place for safety), then blank. `saveSheet` writes only to the character-library key. The `chatId` argument is retained for interface stability but unused in the new path.
- **`loadCharacters` migrates legacy bare `id: "player"` ids to fresh UUIDs on first load.** Every chat used to default to `id: "player"`, which would collide across chats under the new chat-independent key. Migration: each legacy "player" character gets a fresh `char-{Date.now()}-{random}` id, the chat's character list and `mrrp-active-char-{chatId}` pointer update, and the legacy chat-scoped sheet copies forward to `characterKey(newId)`. Each historical chat keeps its own distinct sheet — no data loss, no collision.
- **What this unlocks:** characters become portable across chats. Foundation for any future "import character into this chat" UI; foundation for Steps 2 (IndexedDB) and 3 (Marinara API mirror) per the roadmap at `~/cc-wiki/Roadmap/marinara-character-library-storage-evolution.md`.


## [0.2.1] - 2026-05-06

Sheet-ergonomics, item lifecycle, and multi-system damage release. Twenty-two iterative rounds of fixes against live in-Marinara playtest, all in service of getting D&D 5e and Exalted 3e to feel correct under the new two-tier inventory + state-mutator + bar UX models. The big themes:

- **Inventory is robust end-to-end.** Items survive load (no silent merge drop), agent-added items carry the full dialog field set (slot, damage, attack_attr, attack_proficient, use_effect, consumable, notes, category), the agent's own promptTemplate documents the schema as extension-confirmed, and consumable Use posts a tag the agent reads.
- **Bars finally make sense.** Every bar shows editable current AND max as separate inputs; user-set max overrides ruleset formula/literal so manual adjustments stick; refresh re-syncs both inputs to state after every state-mutator delta (no more stale UI confusing the agent on the next refresh).
- **Damage parser is multi-system.** `parseDamageExpression` handles D&D `NdM[+K] [type]`, Exalted `12L`/`12B`/`12A`/`12dL`/`12 Lethal` (rolls Nd10 → counts 7+ as successes per Exalted 3e), and flat `5 fire`. Weapons, items, and spells all share one parser.
- **Per-ruleset polish.** Exalted header now reads `Type` / `Caste/Aspect`; D&D Feats section enabled with `textOnly` rows; D&D 5e attribute modifiers + skill bonuses + saves are first-class autocalc; conditions tracker with roll-mode aggregation; item categories (equipment vs item) with separate Equipment section + Items flyout panel.



### Added — Sheet ergonomics batch (six features)

- **Editable numeric inputs everywhere.** Attributes, skills, derived stats, backgrounds, custom skills, and bar current values (HP, motes) all swap their value `<span>` for an `<input type="number">`. Type a number and press Enter or tab away to commit; the +/- stepper still nudges. Native browser spinners are suppressed because the stepper already provides that affordance and double controls are visual noise.
- **Identity row in the sheet header.** Two free-text fields below the character switcher whose labels are ruleset-driven: Race + Class (D&D 5e), Ancestry + Class (PF2e), Exalt Type + Caste (Exalted 3e), Concept + Trouble (Fate Core). New optional `header.raceLabel` / `header.classLabel` in the ruleset schema; old bundles fall back to "Race" / "Class".
- **D&D 5e Feats section.** The dnd5e ruleset and bundle now ship with `backgrounds: { enabled: true, label: "Feats", min: 0, max: 5, default: 0 }` and `"backgrounds"` inserted into `sheetSections` between states and inventory. The existing free-text dot-rated trait pattern carries D&D feats with no new section type.
- **Add custom skills / lores.** A "+ Add Skill" button at the foot of the Skills section pushes a fresh entry into `state.sheet.customSkills`. Each row has a free-text name input, optional linked-attribute select drawn from the ruleset's attributes, value display + stepper, and remove × button. Multiple custom skills are supported. Custom skill values participate in `statContext()` so dice formulas and `valueFormula` can reference them by name.
- **Autocalc for derived stats.** New optional `derivedStats[].valueFormula` (parallel to existing `maxFormula`). When present, the sheet displays the computed value and suppresses the manual stepper — the formula IS the override. Same `{StatName}` placeholder + arithmetic-only whitelist as `maxFormula`. Exalted 3e populated for Defense (Parry), Defense (Evasion), Resolve, Guile, Soak (Bashing), and Soak (Lethal).
- **Backwards-compatible storage shape.** New fields (`identity`, `customSkills`) default to empty on `blankSheet`; `mergeSheet` accepts either presence or absence. Old saved sheets load without migration.

### Fixed — v0.2.x Round 22 — Damage parser handles Exalted notation (NL/NB/NA, NdL/NdB/NdA, "N Lethal") (2026-05-06)

- **Damage parser was hardcoded to D&D's `NdM[+K] [type]` regex.** Live test in Exalted surfaced three Exalted-flavored damage strings the agent emitted that the parser rejected: `12L` (12 dice, lethal), `12dL` (dice-pool variant), and `12 Lethal` (word type). All three logged `rollWeaponDamage: cannot parse '...'` and the dmg button silently failed.
- **New unified `parseDamageExpression(s)`** returns a tagged result `{kind: "dnd"|"exalted"|"flat", ...}`. Three regexes feed it: D&D `NdM[+K] [type]`, Exalted single-letter `N[d]X` where `X ∈ {B,L,A}`, and flat `N type-word`. When the type word is Bashing/Lethal/Aggravated the flat branch routes to Exalted (matches Exalted convention that the number is dice pool count, not result). Anything else stays flat.
- **New `rollParsedDamage(parsed, opts)`** dispatches by kind: D&D rolls Nd-size and sums; Exalted rolls Nd10 and counts 7+ as successes (Exalted 3e damage convention — 10s do NOT double on damage rolls); flat posts the value as-is. Returns `{text, faces, kind}` so the caller finalizes via `finalizeRoll`.
- **`rollWeaponDamage`, `useItem`, and `castSpell` (ability.damageDice)** all now route through the unified parser. One contract, one place to extend when a future ruleset wants its own damage notation. Exalted `12L` weapons now roll 12d10 and count successes; `2d4+2 healing` potions still sum dice; `5 fire` flat values just post.
- **`mrrp-dice__face--success` CSS class** already existed; Exalted dice rolls now apply it to faces showing 7+ for visual success-counting feedback.
- **Re-paste the extension JS** to activate. No bundle change required.

### Fixed — v0.2.x Round 21 — Bars always show editable max + refresh syncs inputs + Willpower per-character cap (2026-05-06)

- **All bars now render an editable max input alongside current.** Previously `renderBar` gated the max input behind `!hasExplicitMax`, so Personal Motes / Peripheral Motes / Willpower (all formula- or literal-capped) hid the max — the player could only see "current" with no way to inspect or override the cap, and a state-mutator refresh attempt produced math errors because the agent had only one number to read. Now every bar shows `[current] / [max]` as two editable inputs. For ruleset-capped bars the max input default-displays the formula/literal value; typing a value persists to `state.sheet.derivedMax[name]` and overrides the ruleset (Anima boosts, temporary Willpower modifiers, GM-side tweaks the formula doesn't model). Tooltip on the max input names the formula/default explicitly so it's clear what the user is overriding.
- **`refresh()` now re-syncs both inputs to current state.** A state-mutator delta like `[mrrp-state: field="Personal Motes" delta="+5"]` writes to `state.sheet.derived["Personal Motes"]` and fires `refreshAllBars()`. Pre-fix: the bar fill + label updated, but the editable inputs kept showing stale numbers — the player saw "8" in the input but the bar said "13 / 25", and the GM agent's next refresh attempt computed against the stale display. Fix: refresh() unconditionally writes `inputEl.value` and `maxEl.value` from current state.
- **`computeMax()` derivedMax check moved to top.** User-set override now wins over `maxFormula`/`max` literal so manual edits stick across re-renders. Previously the formula always recomputed and clobbered the user override; now once the user types a max value, that's the cap until they clear it.
- **Willpower no longer has a hardcoded `max: 10` in the ruleset.** Canonical Exalted Willpower caps are per-character (typically 5–10 for Solars). The literal cap forced every character to display "10" as the cap. Removed `max: 10` from the exalted3e Willpower derived-stat definition; the user now sets the cap via the bar's max input. Default for fresh sheets falls through the auto-grow fallback until the user types a value (recommended: 5 for starting Solars).
- **Re-install the exalted3e bundle** to pick up the Willpower ruleset change. Re-paste the `extension/RPG-Extension-RP-Mode.js` to pick up the renderBar + refresh fixes (CSS already embedded by `tools/embed-css.mjs`).

### Added — v0.2.x Round 20 — Exalted header labels + capped-value display (2026-05-06)

- **Exalted 3e header labels updated.** `header.raceLabel: "Exalt Type"` → `"Type"`; `header.classLabel: "Caste"` → `"Caste/Aspect"`. Bundle regenerated; reflects the canonical Solar/Sidereal/Abyssal vocabulary (Exalts have a Type and a Caste; Dragon-Bloods use Aspect — the slash makes it work for both without forcing a single term).
- **Derived stats with declared caps now show "current / max" inline.** `renderValue` (used for `renderAs: "value"` derived stats) gained a `computeValueMax()` helper and an inline `mrrp-row__cap` span that reads `" / N"` whenever the ruleset declared a `max` literal or a `maxFormula`. Essence reads "5 / 10", any value-rendered capped stat reads similarly. The cap label refreshes when bars refresh, so a capped stat keyed on another stat's value (Personal Motes' max keyed on Essence) updates live. Editable input + stepper now also clamp to the computed cap when one is declared. CSS class `.mrrp-row__cap` styled in monospace dim-text matching the existing bonus suffix idiom.
- **`renderAs: "bar"` unchanged** — Willpower (already a bar with `max: 10`) keeps its bar-fill UX with the existing "current / max" label inside the fill. The new cap suffix only adds visibility to value-rendered stats that were previously bare numbers.

### Added — v0.2.x Round 19 — State-mutator agent body documents the full inventory schema (2026-05-06)

- **Agents themselves now know the full schema.** Round 18 wired the parser + helper + lorebook field-reference doc, but the agent's own `promptTemplate` (built from `agents/state-mutator.md` + per-ruleset overrides) still listed only the original 5 fields (`target/field/add/remove/qty/reason`) as canonical, so the agent kept saying "extended fields unconfirmed by parser" and refused to emit them. Updated all six source markdown files (shared `agents/state-mutator.md` + `rulesets/dnd5e/agents/state-mutator.md` + `rulesets/exalted3e/agents/state-mutator.md` per repo) with: (a) the inventory tag protocol line now lists the optional attrs inline; (b) a new `# Inventory schema (full field list — extension-confirmed)` section documents all 8 attrs with examples and the "repeated add enriches blank fields" semantic; (c) two new worked examples (stored consumable, weapon gift between characters) join the existing damage / save / spell-slot examples. The phrase "extension-confirmed" is deliberate — earlier rounds the agent's own diagnostic noted "no parser guarantee" on extended fields; the heading wording lets the agent know these ARE supported.
- **Rebuilt agents.json + bundle.json for all three rulesets per repo.** `tools/build-agents.mjs --all` regenerated `rulesets/{dnd5e,exalted3e,fate-core}/agents.json` from the updated source markdown; `tools/build-bundle.mjs --all` regenerated bundles. (pf2e is bundle-only; agents.json doesn't apply.) All gates green: `node --check`, `validate-rulesets` 3/3 PASS, `validate-bundles` 4/4 PASS.
- **Re-install is required to push the new schema to live agents.** Per the v0.4 architecture, agents are decoupled from the bundle and installed via the "Import Agents" dialog. To activate Round 19 in your live Marinara session: reload the extension JS, then re-import the regenerated `rulesets/dnd5e/agents.json` via the extension's Import Agents flow. The dialog uses delete-then-replace, so the State Mutator overlay's promptTemplate is replaced wholesale with the new schema-aware text.

### Added — v0.2.x Round 18 — State-mutator inventory tags accept full dialog field set (2026-05-06)

- **`[mrrp-state: action="add" field="inventory"]` now accepts dialog attrs.** The Round 17 fix made agent-added items survive load and render correctly, but the agent could only set `name`/`quantity`/`reason` — every other dialog field defaulted to empty. Logs from the live test showed the RP agent itself flagging the gap: "for stat/effect field support on inventory tags. Client parser behavior is unknown." The parser already extracted every `key="value"` pair via `STATE_KV_RE`; only `applyStateMutation` was throwing the rest away. New `applyItemAttrs(item, attrs)` helper maps `slot`, `damage`, `attack_attr` → `attackAttribute`, `attack_proficient` (bool), `use_effect` → `useEffect`, `consumable` (bool), `notes`, and `category` (`"equipment"` or `"item"`) onto the item. Snake-case keys are LLM-friendly inside a tag.
- **Auto-categorize new items by slot presence.** When the agent sends `slot="weapon"` without `category`, the new push order now applies attrs FIRST and normalizes SECOND, so the normalizer's existing slot-presence rule (`slot ? "equipment" : "item"`) infers `category="equipment"` automatically. Explicit `category="..."` always wins over inference.
- **Repeated adds enrich blank fields.** When `[mrrp-state: add="Sword" damage="1d8"]` hits a name-match in inventory, `applyItemAttrs` runs on the existing item before the quantity bump — agent can populate fields once authoritatively on first add, omit them on subsequent qty bumps. Empty strings are ignored (omit a field to leave it alone). Booleans only land on truthy values; once `true`, can't be unset via tag — use the dialog.
- **Field-reference lorebook entry advertises the full schema.** `buildFieldReferenceContent` now emits four worked examples (bare / stored consumable / weapon / armor) plus an 8-row attrs reference table. Auto-sync PATCHes the lorebook entry on next save, so the narrator and overlay agents see the new schema on the next generation without any manual lorebook edit.

### Fixed — v0.2.x Round 17 — Items vanish after edit, agent-added items render bare (2026-05-06)

- **State-mutator inventory.add now produces fully-formed items.** When the RP agent emits `[mrrp-state: action="add" field="inventory" add="Potion"]`, `applyStateMutation` was pushing `{name, quantity, description, location}` only — no `id`, no `category`, no dialog fields. Two downstream effects: (a) the item rendered bare in the Equipment / Items flyout (no slot, damage, bonuses) because the renderer reads fields that didn't exist, and (b) `mergeSheet`'s inventory branch silently dropped every id-less item on the next character switch / page reload. Net: items appeared "with the right name but no stats" and "got deleted any time another item was modified". Fix: state-mutator's add path now routes through a new `normalizeInventoryItem()` helper that fills `id`, `category`, `slot`, `damage`, `attackAttribute`, `attackProficient`, `bonuses`, `useEffect`, `consumable`, and `notes` to safe defaults.
- **mergeSheet self-heals legacy partial items instead of dropping them.** The inventory filter no longer requires `typeof it.id === "string"`. Instead, every loaded item runs through `normalizeInventoryItem(it, idx)`, which assigns a deterministic `item-heal-{nameSlug}-{idx}` id (stable across reloads so `state.sheet.equipped[slot]` references survive) plus all dialog-field defaults. Items already saved to localStorage from any prior session — including the broken pre-fix state-mutator pushes — are upgraded in place on next load, no manual re-add required.
- **State-mutator existing-name merge backfills missing fields.** When `[mrrp-state: action="add"]` hits a name-match in the inventory, the existing item now passes through `normalizeInventoryItem` before the quantity bump. Previously, an existing partial item stayed partial and would throw on `draft.bonuses.forEach(...)` when the user opened it for editing.

### Fixed + Added — v0.2.2 round (2026-05-06)

- **HP / bar manual entry now accepts any value.** Bug: when a derived stat is `renderAs: "bar"` with no `max` or `maxFormula` declared (D&D Hit Points), the input was clamping to `DEFAULT_BAR_MAX = 10`. Fix: when neither cap is declared the input is unclamped on the upper side, the stepper goes up to 9999, and the bar fill auto-grows so it always shows partial progress instead of saturating against a phantom 10.
- **Autocalc now actually computes D&D / PF2e skill bonuses.** Prior batch only handled derived-stat `valueFormula`; skills displayed raw user-entered numbers. New: when a ruleset declares `resolution.skillBonusFormula` (e.g. D&D's `"{linkedAttribute_mod} + {tierBonus}"`), every skill row shows the computed bonus and hides the raw stepper. Concrete behavior: Sleight of Hand at Dex 16, Trained, Level 4 displays **+5** automatically.
- **Attribute modifiers are now a first-class concept.** New optional `attributes[].modifierFormula` field — D&D 5e ships with `({Score} - 10) / 2`. The renderer computes each attribute's modifier on every refresh and exposes it under both `<name>_mod` and `<abbreviation>_mod` (so formulas can write `Dexterity_mod` or `DEX_mod` interchangeably).
- **Level + autocalc'd Proficiency Bonus.** D&D 5e gains a Level derived stat (default 1, max 20). Proficiency Bonus gets `valueFormula: "2 + (({Level} - 1) / 4)"` — automatically advances 2/3/4/5/6 at L1/5/9/13/17. Math.floor (not round) is now used for autocalc display so D&D mod (9-10)/2 = -1 (not 0) and Exalted ceiling formulas remain correct.
- **HP stays manual.** Explicitly: Hit Points has no `valueFormula` — the table decides their own HP method (max-per-level, rolled, average, etc.).
- **Saving Throws section.** New ruleset field `saves: [{ name, linkedAttribute }]` plus a `"saves"` sheetSection key. D&D 5e ships with all six (one per ability), each with proficiency-tier toggle. The displayed save bonus = attribute mod + tier bonus, computed live the same way skills are.
- **Item damage field.** Inventory items gain an optional free-text `damage` field ("1d8 slashing", "2d6 fire"); the item dialog has a Damage row between Slot and Bonuses; the inventory list shows the damage in a warning-hued cell next to the slot tag.
- **System-agnostic guarantee.** Every autocalc path is opt-in per ruleset: Exalted (no `modifierFormula`, no `skillBonusFormula`) keeps using `valueFormula`-on-derived only; D&D layers modifierFormula + skillBonusFormula on top; a future GURPS bundle can declare its own formulas without engine changes. The renderer respects whatever the ruleset provides.

### Fixed (carried forward from prior round)

- Pathfinder 2e bundle gained `attributes[].modifierFormula = "{Score}"` (PF2e attributes are already-modified) plus `Level` derived stat and `resolution.skillBonusFormula = "{linkedAttribute_mod} + {tierBonus}"`. PF2e tier formulas already include `{Level}`.

### Fixed — Agent visibility (LLM couldn't see the sheet)

- **First diagnosis (incomplete):** `syncSheetToChat` was button-triggered only and exported only legacy fields. Fix part 1 added auto-sync on save and a complete payload via new `buildSyncFields(prefix)` (identity, raw attributes, computed attribute mods, skill/save bonuses, custom skills, derived, states, backgrounds).
- **Live test surfaced the deeper bug:** Even with the auto-sync running and 45 fields PATCHed to `chats.customTrackerFields`, the overlay agents still saw zero numbers. **Reading the Marinara engine source revealed why:** `chats.customTrackerFields` only reaches generation context via Marinara's stock `custom-tracker` agent (which writes a snapshot to `gameStateSnapshots.playerStats.customTrackerFields` that the marker-expander then reads). The player's chat does not have the stock `custom-tracker` agent enabled — only the six `mrrp-overlay-v1` agents. So everything we PATCHed into the chat field was visible only to the chat UI tracker panel, not the LLM.
- **Real fix: prompt-template injection into managed agents.** New `buildSheetForPrompt()` renders the live sheet as a markdown block ("LIVE CHARACTER SHEET — Corey (Dungeons & Dragons 5th Edition v1.0.0)…" with identity / attributes-with-mods / skills-with-computed-bonuses / saves / derived / inventory-with-damage / backgrounds). New `syncSheetToAgents()` GETs all agents, filters to managed overlays for the active ruleset, and PATCHes each agent's `promptTemplate` to embed the sheet block between `<!-- MRRP_SHEET_BEGIN -->` / `<!-- MRRP_SHEET_END -->` markers. New `injectSheetIntoPromptTemplate(prompt, block)` strips any prior block before prepending the new one, so updates over time never accumulate stale sheets.
- **Wired to the same debounce.** `scheduleAutoSync()` now fires both `syncSheetToChat` (for the UI tracker panel) AND `syncSheetToAgents` (the LLM-reaching path) on a single 1.5s debounce after every save and on ruleset activation. Six agents × one PATCH each per save burst = manageable round-trip cost on localhost.
- **No re-install required.** Reloading the JS extension is enough — the new sync path PATCHes the existing already-installed agents. Bundles do not need to be re-pasted to fix this.
- **Regression patched (round-5 hot-fix):** `syncSheetToAgents`'s filter was copy-pasted from the GM repo unchanged and looked for `s.mrrManaged === true && s.mrrRulesetId === rulesetId`. RP-mode agents are tagged with `mrrpManaged` / `mrrpRulesetId` per the namespace separation. Console log was the smoking gun: `[mrr] syncSheetToAgents: no managed agents for ruleset dnd5e` fired on every save — meaning the filter rejected all six existing agents, and the PATCH never ran. Filter now uses the correct `mrrp*` keys.

### Added — Player rolls (saves, initiative, attack, damage)

- **`quickRollForSave(save)`** — opens the dice widget pre-filled with mod = linked attribute modifier and prof = active proficiency tier bonus. Each save row now ends with a `roll` button.
- **`quickRollForDerived(derived)`** — opens the dice widget for any derived stat that declares `rollFormula`. D&D 5e Initiative ships with `rollFormula: "{Dexterity_mod}"`. Roll button on both autocalc and manual derived rows when rollFormula is set.
- **`quickRollAttack(item)`** — opens the dice widget pre-filled with 1d20 + attack-attribute modifier + (proficiency bonus when `item.attackProficient`) + equipped item bonuses. Item rows render an `atk` button when `item.attackAttribute` is set.
- **`rollWeaponDamage(item)`** — parses the damage string ("1d8 slashing", "2d6 fire", "1d4+1 piercing"), rolls, adds `attackAttribute_mod` if set, posts `[damage: …]`. Item rows render a `dmg` button when `damage` is set.
- **Item dialog** gains Atk attr dropdown + Proficient checkbox.
- **Schema:** `derivedStats[].rollFormula`; inventory items gain optional `attackAttribute` and `attackProficient`.
- **Skill-roll autocalc fix:** `quickRollForSkill` was filling `mod` from the raw `state.sheet.skills[name]` (always 0 with autocalc). Now reads `<linkedAttribute>_mod` when `skillBonusFormula` is set so dice widget pre-fills correctly.

### Fixed — State-mutator HP and other shorthand fields

- **Bug:** Console log `[mrr] state-mutator: unmatched field 'hp' — stashed on sheet root` fired every turn. The LLM emits `[mrrp-state: field="hp" delta="-X"]` but D&D's canonical derived-stat name is "Hit Points". `resolveSheetField` only matched canonical and case/punctuation-insensitive variants, so "hp" → normalized "hp" never matched "Hit Points" → "hitpoints". Mutation values stashed on the sheet root and the HP bar never moved.
- **Fix:** Added an aliases pass to `resolveSheetField`. Each derived stat / attribute / skill can now declare `aliases: [...]`. Attribute `abbreviation` is auto-recognized so `field="DEX"` resolves to "Dexterity" without needing an explicit alias. D&D 5e ships with: Hit Points → ["hp", "HP", "hitPoints", "hitpoints"], Armor Class → ["ac", "AC", "armorClass"], Initiative → ["init"], Proficiency Bonus → ["prof", "proficiency", "PB"], Speed → ["speed", "movement"], Level → ["level", "lvl", "characterLevel"].
- **Agent self-documentation:** `buildSheetForPrompt` now appends a "State-mutator field reference" block listing every canonical name with its declared aliases. The agent sees this in its prompt every turn and learns which names resolve.

### Added — Triple-coverage field reference (lorebook bridge)

The aliases pass and the agent-prompt block both reach our six overlay agents — but **the main narrator** (the model that actually emits `[mrrp-state: ...]` tags inside the chat narration) does not read overlay agent prompts. It reads the lorebook. So the field reference now lives in three places, and the narrator picks it up via the third:

- **`buildFieldReferenceContent()`** — formats the canonical names + aliases for every `derivedStats` / `attributes` / `skills` entry on the active ruleset, plus a "common compound mutations" cheat sheet (HP delta, condition add/remove, inventory gain).
- **`syncFieldReferenceToLorebook()`** — finds the managed lorebook for the active ruleset, finds (or creates) an entry tagged `mrrp-field-reference`, and PATCHes its content. The entry uses `constant: true` (Marinara's "always include" lorebook flag) so the narrator sees it every turn regardless of keyword triggers. Tagged so we update in place rather than spawn duplicates on every save.
- **Wired to the same debounce.** `scheduleAutoSync()` now fires three sync paths after every save and on activation: chat customTrackerFields (UI tracker panel) → overlay agent promptTemplates (six PATCHes) → managed lorebook field-reference entry (one PATCH).
- **Exalted 3e alias adoption:** Personal Motes / Peripheral Motes / Willpower / Essence / Health Track / Defense (Parry) / Defense (Evasion) / Resolve / Guile / Soak (Bashing) / Soak (Lethal) / Sorcerous Motes — each now declares the LLM's natural shorthands ("motes", "wp", "soakBashing", etc.). Same belt-and-suspenders that D&D got, system-agnostic — any future ruleset (GURPS, Lancer, Cyberpunk RED, …) just declares its own aliases and the triple-coverage applies automatically.

### Fixed — Bar max persistence and dotted-path stash

- **Bug:** Take 15 damage on D&D HP 15 → bar showed "0 / 10" (max collapsed). Cause: `computeMax` used `Math.max(DEFAULT_BAR_MAX, current)` when no engine cap was declared, so dropping current to 0 collapsed the visual max back to 10. The bar's denominator wasn't persistent.
- **Fix:** New `state.sheet.derivedMax` bucket — the user's typed max for bars without a ruleset-declared cap. `computeMax` prefers it over the auto-grow fallback, so the bar's denominator survives any current-value change. `renderBar` now renders a SECOND editable input next to current when no engine cap is declared (D&D HP); a "/" separator sits between them. Engine-capped bars (Exalted Motes/Willpower) hide the max input — engine still controls them via maxFormula.
- **`mergeSheet`** validates `derivedMax` on load; old saves without it default to `{}`.
- **Bonus fix: dotted-path support.** The state-mutator was emitting `field="deathSaves.failures"` and the resolver flattened it onto the sheet root as a single key. Now the stash fallback splits on `.`, walks/creates nested objects, and writes the leaf — so `deathSaves.failures = +1` lands at `sheet.deathSaves.failures` correctly. Sets the floor for future structured-state features (D&D death saves, exhaustion levels, custom trackers) without locking the schema.

### Added — Advantage / disadvantage on d20 rolls

- **Three-way toggle** in the dice widget (single-roll mode only): Normal (1d20), Adv (2d20 keep highest), Dis (2d20 keep lowest). Active mode picks up the accent color. Persists in `state.diceAdvantage` across rolls within a session.
- **Result text shows both faces.** Format: `[dice: 2d20kh1+5+2 vs DC15 = 19 success (kept 19, dropped 7 — advantage)]`. The state-mutator and any GM reading the chat see exactly what landed and what didn't.
- **Wired into existing roll buttons.** `quickRollForSkill`, `quickRollForSave`, `quickRollForDerived`, `quickRollAttack` all open the dice widget — the player picks adv/dis once and rolls.

### Added — Spell cast workflow (DC + damage + lorebook)

- **`Cast` button on every spellbook ability with cast-time data** (damageDice, saveAttribute, or spellcastingAttribute). One click computes the spell save DC, rolls damage, and posts both as chat tags ready for "Send to chat".
- **Auto-computed DC.** Formula is ruleset-driven via `resolution.spellSaveDcFormula`. D&D 5e ships with `"8 + {Proficiency Bonus} + {spellcastingAttribute_mod}"`. PF2e bundle ships with `"10 + {Level} + {spellcastingAttribute_mod}"`. The `{spellcastingAttribute_mod}` token is pre-substituted with the spell's specific caster-attribute mod before evaluation.
- **Cast tag format:** `[mrrp-cast: name="Fireball" dc="15" save="Dexterity" damage="8d6 fire" half_on_save="true" cost="1 lvl-3 slot"]`. Followed by the rolled `[damage: 8d6 = 27 fire (Fireball)]`. The narrator reads both — DC for save resolution, damage to apply post-save.
- **Spell fields** added to the ability dialog: Damage (free-text dice), Cast attr (caster's spellcasting ability), Save vs (defender's save target), Half on save (D&D 5e half-damage default). All optional — non-spell abilities (passives, social charms) leave them blank and the Cast button doesn't render.
- **Lorebook entry** now embeds a "Cast Mechanics" block with all four cast fields. The GM/state-mutator agents read it when the spell's keyword fires in chat — they see exactly which save to call for, the DC, and the half-on-save rule, without having to re-derive from narration.
- **Workflow:** create spell in spellbook → it auto-syncs to the lorebook → click Cast → DC + damage tags drop into the dice widget → "Send to chat" → narrator/GM agent resolves saves and applies damage via the existing `[mrrp-state: field="hp" delta=...]` pipeline.

### Added — Active conditions on the sheet (with mechanical effects)

- **New "Conditions" section** on the character sheet between States and Backgrounds. Lists currently active conditions from `state.sheet.conditions` with × remove buttons; add via a dropdown of ruleset-declared options or a free-text "other" entry.
- **Ruleset-declared condition definitions.** New `ruleset.conditions: [{ name, description, imposesDisadvantageOn?, grantsAdvantageOn? }]` schema field. Each condition can declare which roll categories — `attack`, `save`, `skill` — it imposes disadvantage or grants advantage on.
- **Mechanical effects on rolls.** New `conditionRollMode(category)` helper aggregates every active condition's effect. `quickRollForSkill`, `quickRollForSave`, `quickRollAttack` consult it before opening the dice widget — auto-arming the Adv/Dis toggle. Disadvantage wins ties.
- **Agent visibility.** `buildSheetForPrompt` emits an "Active Conditions" block with each condition's effects + description, so the narrator sees "Poisoned — disadvantage on attack, skill" without the player having to narrate it.
- **D&D 5e bundle adoption.** All 15 SRD conditions ship: Blinded, Charmed, Deafened, Frightened, Grappled, Incapacitated, Invisible, Paralyzed, Petrified, Poisoned, Prone, Restrained, Stunned, Unconscious, Exhausted. Roll-mode mappings encoded for the canonical SRD effects.
- **State-mutator pipeline unchanged.** The existing `[mrrp-state: field="conditions" add="..."]` tags still drive `state.sheet.conditions`; the new layer just visualizes and consults the data.

### Fixed — Two D&D leaks in the core engine (system-agnostic discipline)

- **`castSpell` DC fallback removed.** Now ruleset-driven only — when `spellSaveDcFormula` isn't declared, the cast tag posts without a DC.
- **`quickRollAttack` proficiency lookup generalized.** Was `ctx["Proficiency Bonus"]` (hardcoded stat name); now reads `resolution.attackProficiencyFormula` (D&D 5e bundle declares `"{Proficiency Bonus}"`).
- Zero hardcoded D&D conventions remain in runtime paths. The core extension provides generic engine surface; per-system rules live entirely in the bundle.

### Added — Text-only background entries (D&D Feats)

- **New `backgrounds.textOnly: bool`** schema flag. When true, name-only rows (no dot value, no stepper). D&D 5e Feats use it; Exalted/WoD systems leave it false to keep the dot rating.

### Added — Two-tier inventory: Equipment + Items flyout

- **Item shape:** new optional `category` (`"equipment"` | `"item"`), `useEffect` (dice expression), `consumable` (bool). `mergeSheet` infers category from slot presence on legacy items.
- **Equipment section** filters inventory to category="equipment", renamed visually from "Inventory" to "Equipment".
- **Items flyout** — separate floating panel (mirrors spellbook). "Open items" button at foot of Equipment. Lists items with quantity, useEffect, Use / Edit / Delete buttons.
- **`useItem(item)`** parses useEffect via DAMAGE_RE, rolls, posts `[mrrp-use: name="..." effect="..." rolled="..." consumable="true"]`. Decrements quantity; removes when 0.
- **Item dialog** gains Category / Use effect / Consumable rows.
- **System-agnostic:** Exalted's burner phone (item, no slot) and artifact bracers (equipment, slot=arms) work identically.

## [0.2.0] - 2026-05-06

The first substantive Roleplay Mode release after the v0.0.1 initial publish. Brings the framework to functional parity with the GM Mode sibling at v0.4.0, with additions specifically suited to roleplay-mode (the narrator runs alongside Marinara's default world-state, prose-guardian, continuity, and expression agents instead of replacing the engine's own GM). Ships a self-contained download under `releases/v0.2.0/` with seven AI-feedable build docs, two complete reference rulesets (D&D 5e and Exalted 3e), and drop-in install files. Targeted at users who want to point a chat AI at the documentation and have it author a complete ruleset for any tabletop RPG system.

### Added — Typed damage on tracks

- **`damageTypes` schema field** on `renderAs: "track"` derived stats. Declares a list of damage types (id, label, severity, description). When present, the renderer shows each filled cell colored by type, stacks higher-severity damage to the leftmost positions, and provides per-type "Take" buttons plus a "heal worst" button.
- **State-mutator routes typed damage automatically.** `field="bashing" delta="+3"` mutates `state.sheet.track["Health Track"].bashing`. Mirror-mapped against the active ruleset's declared damage type ids.
- **CSS modifier classes** `.mrrp-track__cell--<typeId>` for each declared type. Bashing yellow, Lethal red, Aggravated dark maroon in the bundled Exalted ruleset; system authors define their own palette.
- **Migration:** legacy single-counter `track[name] = N` data folds into `bashing` (the lightest type) on first read so existing characters auto-upgrade without data loss.

### Added — Sorcery / multi-turn casting

- **`sorcery` ability category** is special-cased by the spellbook → lorebook push. Entries get a `Type: Sorcery` content header and a `sorcery` keyword, signaling the state-mutator to use multi-turn shape-sorcery flow instead of immediate-cost charm flow.
- **`Sorcerous Motes` derived stat + `Sorcery Circle` / `Shaping Spell` states** added to the Exalted reference ruleset; documented as the system-agnostic pattern for any RPG with multi-turn casting.
- **State-mutator override walks the casting workflow:** declare → roll → accumulate → unleash with Willpower refund, plus leak-on-no-action and abort-on-switch handling. Documented in `releases/v0.2.0/docs-for-ai/05-AGENT-AUTHORING.md`.

### Added — Build pipeline separation

- **`tools/build-agents.mjs`** — generates `rulesets/<system>/agents.json` from per-system overrides + shared baseline `agents/*.md`. The `main` role comes from `gm-agent.md`; sub-agent roles fall back to the shared baseline when no per-system override exists.
- **Per-system agent override pattern.** Drop a `.md` at `rulesets/<system>/agents/<role>.md` to tune any role for that system; the build tool prefers the override over the shared baseline. System-agnostic by default, system-specific by exception.
- **Bundle no longer carries agents.** `tools/build-bundle.mjs` produces `bundle.json` with ruleset + lorebook only. Agents install through Marinara's Import Agents dialog as a separate operation. Decoupling lets prompt updates ship without forcing users to reinstall the entire ruleset.

### Added — Documentation for AI-assisted authoring

- **`releases/v0.2.0/docs-for-ai/`** — seven numbered Markdown files designed to be fed to a chat AI (ChatGPT / Claude.ai / Gemini) so the AI can author a complete ruleset for any system. Covers overview, agent architecture, schema, lorebook format, agent prompt-writing patterns, build pipeline, and copy-paste prompts for AI authoring.
- **`releases/v0.2.0/examples/`** — two complete worked examples (D&D 5e and Exalted 3e) that AI authors can pattern-match against.
- **`releases/v0.2.0/install-files/`** — the framework JS plus pre-built `bundle.json` and `agents.json` for both example systems. Three pastes from zero to playing.
- **`releases/v0.2.0/BUILD-YOUR-OWN-RULESET.md`** — three options for non-technical users (AI-with-repo, AI-with-zip, manual).

### Fixed — Critical correctness bugs

- **CSP-safe formula evaluator** — the framework's `{StatName}*N+M` parser uses a hand-rolled recursive-descent evaluator instead of `new Function`. Marinara's page CSP blocks `'unsafe-eval'` in many configurations; every formula was throwing silently and bar maxes were falling back to `DEFAULT_BAR_MAX = 10`. The Personal Motes bar on an Essence-7 Exalt was showing /10 instead of /31. Now correctly evaluates with parens, unary +/-, integer and decimal literals.
- **State-mutator field-name normalization** — `peripheralMotes` resolves to schema's `Peripheral Motes`, `hp` to `HP`, etc. Lowercase + strip whitespace/underscore/hyphen on both sides of the comparison. Eliminates the failure mode where AI-emitted variants got stashed as ghost root keys.
- **Max-clamp on numeric deltas** — `field="Personal Motes" delta="+999"` caps at the formula-computed maximum instead of overflowing. Refresh-to-full works correctly. Resolver consults `derived.maxFormula` (evaluated against live stat context) or static `derived.max`.
- **Persisted state-mutator dedupe** — `processedMessageIds` now persists per-chat in `localStorage["mrrp-processed-msgs-<chatId>"]`. Hard refresh no longer replays every historic mutation. Initial DOM sweep on load only re-wraps tag spans for hidden display; mutations that have already applied don't fire again.
- **Lorebook install rewritten** — the bulk endpoint was returning OK but landing zero entries. Replaced with per-entry POST loop matching the proven spellbook write path. Re-install now does delete-then-add to wipe stale entries instead of accumulating duplicates. `tag: "..."` (singular string) replaced with `tags: [...]` (plural array) on each entry to match Marinara's API expectation.
- **Floating panel position clamp on load** — saved positions outside the current viewport now clamp into bounds on restore instead of leaving the sheet stranded offscreen. The drag handler already clamped at move time; the load path now matches.
- **`saveSheet()` no-args call fixed** — `finalizeMutation` was calling `saveSheet()` with zero args, hitting the early-return guard and never persisting. Now correctly passes `(state.chatId, state.sheet)`.
- **`applyStateMutation` numeric path looked at `sheet.bars` (doesn't exist)** instead of `sheet.derived` / `sheet.attributes` / `sheet.skills`. Every numeric delta was writing ghost top-level keys. New `resolveSheetField()` walks the typed maps; legacy fallback path warns instead of silently corrupting state.

### Fixed — UX

- **Health track grouping.** Adding `-0` / `-1` / `-2` health levels via the Ox-Body buttons no longer appends them after Incapacitated. Penalty-descending stable sort groups like-with-like.
- **Agent prompt FORBIDDEN section** in the Exalted state-mutator override calls out the field-name traps the model used to invent. Pairs with a new `state-reminder` per-system override that surfaces canonical field names every turn.
- **Bundle gmAgent prompt openings refreshed** to remove "Marinara Engine's Game Mode" and "the main GM model" framing. Now opens with cooperative framing: "You provide rules guidance for ⟨system⟩ in Marinara Engine's roleplay mode, working alongside the engine's default world-state, prose-guardian, continuity, and expression agents."
- **`gm-agent.md` source files** had the engine reputation-50-char workaround paragraph removed — that constraint applies only to Game Mode, not the roleplay mode this overlay targets.

### Schema

- `ruleset.json` derivedStats with `renderAs: "track"` may declare an optional `damageTypes: [{id, label, severity, description}]` array.
- `ruleset.json` `abilities.categories` may include `{ "id": "sorcery", "label": "Sorceries" }` to enable the multi-turn casting flow on lorebook push.
- Bundle schema unchanged top-level (still `mrrp-bundle` v1).
- Agent collection schema is `mrrp-agents` v1 — new in v0.2.

### Engineering

- Chat-message `MutationObserver` no longer watches `characterData` mutations.
- Chat-message debounce swapped from raw `setTimeout`/`clearTimeout` to `marinara.setTimeout` + monotonic-token cancel pattern.

### Removed

- The bundle's `additionalAgents[]` field is no longer authored by humans. Existing bundles with `additionalAgents` still install correctly under v0.0.1 framework JS; with v0.2 framework JS the install path skips the bundle's agents and expects the user to import agents separately.

## [0.0.1] - 2026-04-30

Initial roleplay-mode publish. Forked the GM-mode framework, swapped the
`mrr-` prefix for `mrrp-` so both extensions can coexist on the same Marinara
install, recast the GM-Agent prompts as cooperative narrator agents that run
alongside Marinara's default narration agents instead of replacing them.
Initial bundles for D&D 5e and Exalted 3e.

## [Unreleased]

## [0.0.1] — 2026-05-01

Initial scaffold. Forked from `Marinara-RPG-Extension` v0.3.0 to target
Marinara Engine's **Roleplay Mode** (`chatMode: "roleplay"`) instead of
Game Mode (`chatMode: "game"`).

### Added
- Bundle envelope discriminator: schema string is now `mrrp-bundle`. The
  installer matches on this string and rejects bundles authored for any
  other Marinara overlay.
- `mrrp-` namespace across the framework — localStorage keys, agent
  type, agent settings flags, lorebook tags, CSS classes, embed style
  id, console debug surface, and bundle prompt prefix all now use
  `mrrp-` so this overlay can coexist on the same Marinara instance as
  the Game-Mode extension without collision.
- Two reference rulesets ported from the Game-Mode extension: D&D 5e
  (SRD 5.1) and Exalted 3rd Edition.

### Architectural changes from Game-Mode extension
- Targets Marinara's roleplay mode. Roleplay mode default agents are
  `world-state`, `prose-guardian`, `continuity`, `expression` — none of
  Game Mode's `game-master`, `combat`, `party-player`, or `quest`.
- No engine-imposed combat encounter modal, no d20-tag prompt format,
  no 50-character reputation `action` cap. The overlay owns all dice
  math and rules adjudication via the in-extension dice widget.
- Custom agents (`type: "mrrp-overlay-v1"`) dispatch in roleplay mode
  the same as in game mode — verified by reading
  `agent-pipeline.ts` / `agent-executor.ts` directly. Engine does not
  filter custom-typed agents by `chatMode`.

### Known limitations / next steps for v0.1
- Agent prompts in `rulesets/*/gm-agent.md` were ported with engine
  framing intact; they refer to "Game Mode" in places and include the
  reputation 50-char-cap workaround. These will be refreshed in a
  follow-up to drop game-mode-specific guidance and frame as roleplay
  cooperative-context-injection alongside the engine's default
  roleplay agents (world-state, prose-guardian, continuity).
- No browser-tested install yet against a running Marinara at
  `localhost:7860`. Public push deferred until that test passes.
- v0.0.1 ships a single overlay agent per ruleset. Multi-agent
  ambitions (state-tracker agent, lore-lookup agent) deferred to v0.1+.

[Unreleased]: https://github.com/Kenhito/Marinara-RPG-RP-Mode-Extension/compare/v0.0.1...HEAD
[0.0.1]: https://github.com/Kenhito/Marinara-RPG-RP-Mode-Extension/releases/tag/v0.0.1
