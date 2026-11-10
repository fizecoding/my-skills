---
description: Grill me into a complete, validated OpenSpec change (proposal + specs + design + tasks)
argument-hint: "[change-name | refine]"
---
Use the openspec-grill-proposal skill to drive this session.

Arguments: ${@:-}

Interpret the argument:
- empty → interactively decide new vs refine, then proceed.
- "refine" → refine an existing active change: run `openspec list --json`, show
  me the top few active changes, let me pick one, and revise its artifacts in place.
- any other value → treat it as the proposed kebab-case name for a NEW change.

Follow the full skill workflow:

1. Run openspec-scout and codegraph-scout in parallel to build a baseline before
   asking me anything.
2. Grill me in rounds (full frontier each round, single question for high-risk
   decisions) through four phases: proposal → specs → design → tasks.
3. Create the change directory (new mode), then write and validate each artifact
   in order, gating each phase on `openspec validate --change <name> --strict`.
4. After all four artifacts are done and validate passes, run an independent
   reviewer and bring any material findings back to me as new questions.
5. Deliver the complete validated change and offer the apply handoff — but do NOT
   start implementation in this workflow.

Respond in my language. Do not run `openspec init` or `codegraph init`
automatically. Use `openspec status/instructions --json` for all artifact paths.
