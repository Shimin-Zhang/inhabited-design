# NOTICE

inhabited-design-vs
Copyright 2026 Shimin Zhang

This product is licensed under the Apache License, Version 2.0 (the "License").
You may obtain a copy of the License in the `LICENSE` file in this repository or at
http://www.apache.org/licenses/LICENSE-2.0.

## Attribution

`inhabited-design-vs` is original work by Shimin Zhang. It is a [Claude Code](https://claude.com/claude-code)
skill and follows the Agent Skills format (a `SKILL.md` orchestration map plus
supporting protocol files under `references/`).

The pipeline operationalizes the **Verbalized Sampling (VS)** method — enumerate a
candidate pool, verbalize each candidate's fit or failure against the brief, then
draw a deterministic sample from the surviving pool — and the related **Inference-time
Family Guidance (IFG)** framing. These methods inform the design of this skill; no
third-party source code is included or redistributed here.

## Inspirations

`inhabited-design-vs` is an independent work that draws conceptual inspiration from
the following skills. No source code or assets from these projects are bundled or
redistributed here; they are acknowledged as design influences.

- **frontend-design** — Anthropic's Agent Skill for distinctive, production-grade
  frontend interfaces. The notion of committing to craft and avoiding generic
  AI aesthetics informs this skill's adversarial-critic and anti-defaults gates.
  Original work: https://github.com/anthropics/skills/tree/main/skills/frontend-design
  Copyright 2025 Anthropic, PBC. Licensed under the Apache License, Version 2.0.

- **impeccable** by Paul Bakaus — a design-fluency skill that extends Anthropic's
  frontend-design with domain-specific references and an anti-pattern catalog. Its
  emphasis on committed design choices and curated anti-patterns informs this skill's
  typography-VS and slop-gate disciplines.
  Original work: https://github.com/pbakaus/impeccable
  Copyright 2025-2026 Paul Bakaus. Licensed under the Apache License, Version 2.0.

No other third-party code or assets are bundled with this skill.
