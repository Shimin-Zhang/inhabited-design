---
name: inhabited-design
description: Builds polished, target-user-rated web apps/pages/tools/assets via Verbalized Sampling at every input — framing, designer voice, seeds, constraint, competitors, typography. Use when you want every design choice to trace to a deterministic random sample drawn from an enumerated pool rather than model reach. Triggers on phrases like "iterate to a perfect score," "no canon," "really diversify the inspiration," "use IFG/VS," or explicit "use inhabited-design."
user-invocable: true
argument-hint: "[auto]"
license: Apache 2.0. See NOTICE.md for attribution.
---

# inhabited-design

A pipeline that turns "build me X for Y" into something that hits target-user-rated perfect scores, with every input derived through Verbalized Sampling — the model enumerates candidates, verbalizes why each fits or fails the brief, then samples deterministically from the surviving pool via an external Bash gate. The framing sentence is the master conditioning variable: every downstream VS step (designer, seeds, constraint, competitors, typography) is conditioned on it, so variance is coherent rather than noisy. All depth lives in the protocol files in `references/`; this file is the orchestration map.

## When to use

- You want every design choice to trace to a deterministic random sample, not model reach
- You want full auditability: enumerated pool + verbalized reason per entry + millisecond seed logged in `sampling.md`
- The brief is high-stakes enough that the user wants to see and optionally veto each VS pick before it conditions downstream steps
- You want explicit competitor analysis as a formal constraint, not just domain lookups
- Typography is contentious in this brief — the VS process surfaces discipline-specific choices a reflex-ban list can't

## When NOT to use

- One-off prototype, not shipping; the 14-step setup overhead isn't worth it
- The brief doesn't have aesthetic requirements (correctness matters, not craft)
- The user says "just get it working" or has no ICP specificity
- Budget is tight; VS steps add 2–4 extra setup turns before the first build token

## Loop budget

≈200k tokens per iteration (critic cycle + ICP cycle each count). VS setup adds 3–5k tokens per step before the first build. Plan for 3–5 iterations to convergence. Confirm budget tolerance with the user upfront — stricter rubrics or highly constrained framings can extend convergence. This is a convergence pipeline for things worth shipping, not a quick-prototype path.

## Setup

Read `inhabited.md` (or run the teach flow to write one), confirm the product register in one phrase model-side, set mode via `/inhabited-design auto` for non-interactive execution (default is interactive). ICP specificity is the strongest reproducibility lever — "designer for a fitness app targeting first-time marathoners in the Midwest" produces tighter output than "designer for fitness apps."

## The six VS-sampled inputs

Six inputs are generated via Verbalized Sampling and locked to `sampling.md` before any persona or design code is written. Each VS step: enumerate a pool conditioned on context, verbalize fit/fail per candidate, brief-fitness filter, ms-sample via external Bash gate, lock to file.

| VS step | What's sampled | Conditions on | Anchors |
|---|---|---|---|
| Steps 2–4 (Framing VS) | Framing sentence | App X + ICP + register | All downstream VS steps |
| Step 5 (Designer VS) | Primary practitioner | Sampled framing | Designer persona voice + 1 signature work |
| Step 6 (Seeds VS) | 2 inspiration seeds | Framing + sampled designer | 2 bank references |
| Step 7 (Constraint VS) | Hard constraint (1–2) | Sampled framing | Critic's binary constraint gate |
| Step 8 (Competitors VS) | 2 competitors + converge/diverge tables | App X + ICP | Critic's `fits competitor judgment` factor |
| Step 10 (Typography VS) | Primary typeface | Framing + sampled designer voice | Designer persona typographic gate |

Steps 9 (Domain research VS) and 11 (Anti-defaults research) contribute filtered pools to the inspiration bank and `claude_anti_defaults.md` respectively — same VS discipline, no single ms-sample pick.

## The pipeline

14 steps in order. **Step 1 runs in the main conversation** (it uses `AskUserQuestion`). **Steps 2–14 run in dispatched subagents** — the main conversation dispatches, receives the subagent's return artifact, handles present+confirm with the user (interactive mode), then dispatches the next step. The **(present + confirm)** / **(model-side)** tags below describe what happens AFTER the subagent returns. **Each step's full discipline lives in the linked protocol file; the dispatched subagent reads it before executing.**

**Pipeline status (resume).** Before Step 1, `Read` `pipeline_status.md` at the project root if it exists. If any steps are `done`, resume from the first `pending`/`in_progress`/`failed` step rather than re-running prior steps. Mark each step `in_progress` before beginning its work and `done` on completion. The file is at the project root alongside `inhabited.md` and `sampling.md`. Full discipline: `pipeline-status-protocol.md`.

1. **Teach gate** — read `inhabited.md` or run the teach flow. (main convo) → `teach-protocol.md`
2. **Framings VS** — read `framing-poles.md` (11 style-pole quotas, sci/instrument capped at 2/40 ≈ 5%); **generate ≈40 framings to quota** (no curated bank). (present + confirm) → `framing-protocol.md`
3. **Brief-fitness gate** — marketing-director filter on the ≈40 pool + quota self-check (≤2/40 scientific/instrument). (model-side) → `framing-protocol.md`
4. **ms-sample framing** — external Bash gate; stdout `SEED=...` + `PICKS=...` is the only source of truth; lock to `sampling.md`. (model-side) → `sampling-protocol.md` §The external sampling gate
5. **Designer VS** — 10 discipline categories × 2 candidates each (framing-fit + non-canon, generated to discipline rosters; no curated bank); brief-fitness gate (≤3 rounds); ms-sample 1. (present + confirm) → `designer-vs-protocol.md`
6. **Seeds VS** — 10 artifact categories × 4 candidates each; brief-fitness gate; ms-sample 2 seeds. (present + confirm) → `seeds-vs-protocol.md`
7. **Constraint VS** — enumerate 6–8 candidates live under the framing; filter to 3–4; ms-sample 1 (or 2 if the framing is otherwise generic). (present + confirm) → `sampling-protocol.md` §Constraint VS
8. **Competitors VS** — 10 positioning categories × 2 candidates each; ms-sample 2; converge/diverge judgment table per competitor. Skipping the table caps the critic's `fits competitor judgment` at 0. (present + confirm) → `competitor-vs-protocol.md`
9. **Domain research VS** — adjacent-domain refs with brief-fitness filter; 3–5 added to the inspiration bank. (present + confirm) → `domain-research-vs-protocol.md`
10. **Typography VS** — typeface candidates verbalized against 3-word brand voice + physical-object anchor; banned-canon list rejects + repools (only model-side resample exception). (present + confirm) → `typography-vs-protocol.md`
11. **Anti-defaults research** — 3–6 WebSearch queries on current AI-design tells; distill 8–15 patterns into `claude_anti_defaults.md`. Hard-fails if WebSearch unavailable. (model-side) → `anti-default-protocol.md`
12. **Designer persona + per-axis output protocol** — generate `claude_designer.md` in the sampled practitioner's voice; lineage block mandatory; 10 output axes via separate sub-passes with restricted context per axis, logged to `borrowings_v{N}.md` as an attribute-binding manifest. Also generate `claude_icp.md` and `claude_critic.md`. (model-side) → `designer-persona-template.md`, `axis-decomposition-protocol.md`
13. **Critic loop** — 4-factor rubric (fits framing / fits sampled inputs / fits competitor judgment / fits axis manifestation) until 10/10 + constraint honored + slop gate PASS + state-coverage gate. Max 5 cycles. (model-side) → `adversarial-critic-protocol.md`
14. **ICP loop** — 7-factor rubric (Usability, Flare, Personality, Visual Design, Originality, Visual Impressiveness, Wow Factor) until all factors at max; **image fill at the end of each iteration** (stock only — Unsplash/Pexels) so the ICP reviews real images every pass. (model-side) → `iteration-loop.md`, `scoring-rubric.md`

`sampling.md` is the session's audit trail: enumerated pool + verbalized reasoning + ms-seed + locked pick, per VS step. Subagents read it before each major decision; picks are not negotiable mid-loop without explicit user authorization.

## Subagent dispatch

Steps 2–14 each run in a dispatched subagent. The main conversation is the orchestrator:

1. **Read state from disk** — `inhabited.md`, `sampling.md`, `pipeline_status.md`, and any persona / borrowings files written by prior subagents.
2. **Dispatch the next step's subagent** with: (a) pointer to the step's protocol file in `references/`, (b) pointer to the current state files, (c) the present+confirm requirement (interactive mode), (d) the artifact paths the subagent must produce + the `pipeline_status.md` row it must mark `done`.
3. **Receive the subagent's return** — a short summary listing what was written and where.
4. **Present + confirm** with the user (interactive mode) — the subagent's locked pick + justification.
5. **On accept**, dispatch the next step. **On `resample` / `choose N` / `new options`**, re-dispatch the same step with the user action carried into the subagent prompt.

**State persistence.** Subagents share state ONLY via files (`sampling.md`, `inhabited.md`, `pipeline_status.md`, `claude_*.md`, `borrowings_*.md`, `references/`). The main convo never holds step-specific state — it re-reads from disk after each subagent returns.

**Pipeline-status integration.** Each subagent owns its `pipeline_status.md` row transitions (`in_progress` → `done`) per `pipeline-status-protocol.md`. The main convo verifies the row was marked `done` after each subagent returns; if not, the step is incomplete and must re-dispatch.

In auto mode, the main convo skips the present+confirm pause and dispatches the next subagent immediately on return.

## Image fill (per iteration)

Photography is filled **at the end of each design iteration**, not in a separate post-loop phase. Within an iteration the Designer builds with `IMAGE_INTENT` placeholders (holding aspect ratio + encoded intent), then fills every placeholder before handing off — so the ICP always reviews **real images in place** and image↔design mismatches surface every pass, not only at the very end. The trade-off is re-sourcing changed slots each iteration; the gain is the ICP's 7-factor review always covers the real artifact.

- **Source: stock only — Unsplash and Pexels.** Search both with the originality + appropriateness gates (never the first result; serve the ICP's reality). **No AI image generation; no other stock providers.**
- **Re-source on change:** carry filled images forward between iterations; re-fill a slot only when its layout, palette, or intent changes. Filled images obey the file-size guard (≤1600px longest edge; png/jpg/jpeg/gif/webp only).

Full protocol: `inspiration-bank-protocol.md` §Per-iteration image fill.

## Hard constraints

Every gate listed in the pipeline is enforced per-session — violation blocks the downstream step, not a warning. The highest-value gates to keep in working memory:

- **Teach gate** — no sampling, no persona, no design code until `inhabited.md` is confirmed complete. → `teach-protocol.md`
- **Sampling gate** — every sampled input has VS provenance AND executes via external Bash `python3 -c "..."` whose stdout is the only source of truth. The model is forbidden from computing picks internally. Any audit reader can replay the seed with `random.Random(seed).sample(range(N), K)`. → `sampling-protocol.md`
- **Critic gate** — all 4 factors at 10/10 + constraint honored + slop gate PASS + state-coverage gate before the ICP loop starts. Max 5 cycles before escalating. → `adversarial-critic-protocol.md`
- **Pixel verification gate (Jesse Vincent gate)** — both ICP and Designer subagents capture viewport-bounded screenshots (`browser_resize 1280 800` → `browser_take_screenshot filename=...` → `Read` the saved file) at every review. DOM-only review is blocked. → `iteration-loop.md`
- **Image file size guard** — every downloaded or generated reference ≤1600px on longest edge; only png/jpg/jpeg/gif/webp formats saved. → `file-size-guards.md`

Other gates (inspiration-bank composition, anti-default research, typography-VS, axis decomposition, borrowing summary, palette borrowing) are enforced per their step's protocol file. `claude_anti_defaults.md` (researched per-project) is the moving target for "what to avoid" — there is no hardcoded technical floor; the anti-defaults research surfaces whatever is currently overdone.

## Auto mode

Invoke as `/inhabited-design auto`. VS Steps 2–10 run non-interactively — candidates enumerated, reasoning verbalized, ms-sample picks logged to `sampling.md` without confirmation pauses. Output is identical to interactive mode in VS provenance and determinism. Users can interrupt at any point and switch to interactive for remaining steps; document the switch in `sampling.md`. Full procedure: `sampling-protocol.md` §Auto mode.

## Common pitfalls

| Pitfall | Fix |
|---|---|
| Model tries to re-roll a sampled pick because it feels wrong | Only the user can authorize a re-sample. "Feels wrong" = "didn't match trained attractor" = working as intended. |
| Long-tail typography pick includes a banned-canon family | Reject + repool. Only model-side resample exception in the VS pipeline. → `typography-vs-protocol.md` |
| Domain research and competitors blur into each other | Domain = adjacent-domain visual refs; competitors = direct market rivals. Different roles, different borrowing logic. → `domain-research-vs-protocol.md` |
| Designer skips the converge/diverge table for competitors | Critic's `fits competitor judgment` caps at 0. No partial credit. → `competitor-vs-protocol.md` |
| Designer requests +2 domain refs without a justified gap | Reject. Additional refs in later iterations must cite specific iteration feedback (a direction the current bank doesn't cover). |
| Sampled framing fails brief-fitness gate 3+ times in a row | Re-evaluate the brief itself, not the framing pool. → `framing-protocol.md` |
| Outputs cluster on cream + warm + serif despite VS | Check: was `sampling.md` written with VS provenance before the Designer persona generated? Did the framing step force style-pole diversity? Did the critic converge on `fits sampled inputs` = 10/10? If yes to all and clustering persists, add the cluster to the next anti-defaults research. |

## Reference files

21 files in `references/`. Each is the source of truth for its discipline; read before executing the step that depends on it.

| File | Role |
|---|---|
| `pipeline-status-protocol.md` | `pipeline_status.md` format + per-step completion criteria + read/write/resume discipline |
| `teach-protocol.md` | App X / ICP / brand context capture into `inhabited.md` |
| `framing-protocol.md` | Generate-≈40-to-pole-quota + scientific cap + ban-list + Bash ms-sample 1 |
| `framing-poles.md` | 11 style-pole quotas + scientific cap + ban-list + reach-beyond mandate (replaces the 200-entry bank) |
| `sampling-protocol.md` | External Bash ms-sample gate, present+confirm pattern, auto mode, Constraint VS |
| `designer-vs-protocol.md` | 10 discipline categories × 2 candidates (fit + non-canon), brief-fitness gate, roster-based generation |
| `designer-disciplines.md` | 10 discipline reach-into rosters + graphic-design anti-canon ban-list (replaces the ~100-entry bank) |
| `seeds-vs-protocol.md` | 10 artifact categories × 4 candidates, sample 2, borrowing-productivity test |
| `competitor-vs-protocol.md` | 10 positioning categories × 2 candidates, sample 2, converge/diverge tables |
| `domain-research-vs-protocol.md` | Adjacent-domain VS, brief-fitness filter, iteration re-sample rule |
| `typography-vs-protocol.md` | Typeface VS + banned-canon list + repool exception |
| `anti-default-protocol.md` | WebSearch for current AI-design tells; distill to `claude_anti_defaults.md` |
| `designer-persona-template.md` | Persona in sampled practitioner voice; lineage block; 3 hard gates |
| `icp-persona-template.md` | ICP persona as honest, non-charitable target-user reviewer |
| `adversarial-critic-protocol.md` | 4-factor critic rubric, slop gate, state-coverage gate, persona-discipline gate |
| `borrowing-summary-protocol.md` | Named-element borrowing log + per-competitor judgment tables + attribute-binding manifest |
| `axis-decomposition-protocol.md` | 10 output fields via separate sub-passes with restricted context per axis |
| `inspiration-bank-protocol.md` | 7-anchor bank rules, single-image constraint, palette gate, per-iteration image fill (stock: Unsplash/Pexels) |
| `iteration-loop.md` | Critic + ICP loop file conventions, state-coverage gate, pixel-and-DOM discipline |
| `scoring-rubric.md` | 7 fixed factors for the ICP loop (not per-project modifiable) |
| `file-size-guards.md` | ≤1600px longest edge, png/jpg/jpeg/gif/webp only, downscale recipe |
