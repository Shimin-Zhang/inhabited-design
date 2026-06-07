# inhabited-design-vs

A [Claude Code](https://claude.com/claude-code) Agent Skill that turns *"build me X for
Y"* into a polished, target-user-rated web app, page, tool, or asset — where **every
design choice traces to a deterministic random sample drawn from an enumerated pool**,
not to model reach.

Each design input — framing, designer voice, inspiration seeds, hard constraint,
competitors, typography — is generated through **Verbalized Sampling (VS)**: the model
enumerates a candidate pool conditioned on the brief, verbalizes why each candidate fits
or fails, filters to the survivors, then draws a sample deterministically via an external
Bash gate. The framing sentence is the master conditioning variable, so the resulting
variance is coherent rather than noisy. Every pick is logged with its enumerated pool,
per-candidate reasoning, and millisecond RNG seed to `sampling.md` — fully auditable and
replayable with `random.Random(seed).sample(range(N), K)`.

## Why

Generative design tools collapse toward a trained attractor — the same cream-and-serif,
the same safe layout. This skill forces variance to come from a logged, replayable sample
instead of the model's default, and makes every borrowed element name the specific
sampled input it derived from. The result is output that reads as authored, not generic,
and an audit trail that proves it.

## How it works

A 14-step pipeline. Step 1 runs in the main conversation; Steps 2–14 run in dispatched
subagents that share state only through files. Six inputs are VS-sampled and locked to
`sampling.md` before any persona or design code is written:

| VS step | What's sampled |
|---|---|
| Framing (Steps 2–4) | The framing sentence — the master conditioning variable |
| Designer (Step 5) | Primary practitioner whose voice the build adopts |
| Seeds (Step 6) | Two inspiration seeds |
| Constraint (Step 7) | One or two hard constraints the critic enforces |
| Competitors (Step 8) | Two competitors plus converge/diverge judgment tables |
| Typography (Step 10) | Primary typeface, against a banned-canon list |

The build then runs through an adversarial **critic loop** (4-factor rubric: fits framing
/ fits sampled inputs / fits competitor judgment / fits axis manifestation, plus a slop
gate and constraint gate) and an **ICP loop** (7-factor target-user rubric: Usability,
Flare, Personality, Visual Design, Originality, Visual Impressiveness, Wow Factor) until
both converge. Photography is filled from stock (Unsplash/Pexels) at the end of every
iteration so the reviewer always sees real images in place.

Full depth lives in the protocol files under [`references/`](references/); `SKILL.md` is
the orchestration map.

## Installation

This is a Claude Code Agent Skill. Place the directory where Claude Code discovers skills:

```bash
# personal skills
cp -r inhabited-design-vs ~/.claude/skills/

# or, per-project
cp -r inhabited-design-vs <your-project>/.claude/skills/
```

Then invoke it in a Claude Code session (see triggers below). Run
`/inhabited-design-vs auto` for non-interactive execution; the default is interactive,
pausing to present and confirm each VS pick.

## When to use

- You want every design choice to trace to a deterministic random sample, not model reach
- You want full auditability: enumerated pool + verbalized reason + ms-seed per VS step
- The brief is high-stakes enough that you want to see and optionally veto each pick
- You want explicit competitor analysis as a formal constraint
- Typography is contentious and a reflex-ban list isn't enough

It triggers on phrases like *"iterate to a perfect score," "no canon," "really diversify
the inspiration," "use IFG/VS,"* or an explicit *"use inhabited-design-vs."*

## When not to use

- One-off prototype, not shipping — the 14-step setup isn't worth it
- The brief has no aesthetic requirements (correctness matters, not craft)
- You just want it working, or have no ICP specificity
- Budget is tight — plan ≈200k tokens per iteration and 3–5 iterations to converge

## Repository layout

```
SKILL.md            Orchestration map — the 14-step pipeline and all hard gates
references/         21 protocol files, one per discipline (the source of truth each step reads)
LICENSE             Apache License 2.0
NOTICE.md           Attribution and inspirations
```

## Acknowledgments

This skill is original work that draws conceptual inspiration from two frontend-design
skills (both Apache 2.0). No code from either is bundled here — see `NOTICE.md` for full
attribution.

- [**frontend-design**](https://github.com/anthropics/skills/tree/main/skills/frontend-design)
  by Anthropic, PBC — committing to craft and avoiding generic AI aesthetics.
- [**impeccable**](https://github.com/pbakaus/impeccable) by Paul Bakaus — committed
  design choices and a curated anti-pattern catalog.

It builds on the **Verbalized Sampling (VS)** method and the related **Inference-time
Family Guidance (IFG)** framing.

## License

Licensed under the Apache License, Version 2.0. See [`LICENSE`](LICENSE) and
[`NOTICE.md`](NOTICE.md).

Copyright 2026 Shimin Zhang.
