---
name: openspec-grill-proposal
description: >-
  Grill the user through a rigorous interview to produce a COMPLETE, validated
  OpenSpec change — proposal, specs, design, and tasks — ready for apply.
  Use when the user wants to create or refine an OpenSpec change, says
  "grill this", "打磨/评审 change/proposal", "opsx 访谈", or runs
  /skill:openspec-grill-proposal. Do NOT trigger for a plain "summarize this"
  request — the intent must be to create or sharpen a complete OpenSpec change
  ready for implementation.
compatibility: Requires the openspec CLI (https://openspec.ai) with an
  initialized OpenSpec workspace in the project; a CodeGraph index is optional
  (codegraph-scout falls back to grep/read).
---

# OpenSpec Grill-Change Workflow

Requires the `openspec` CLI and an initialized OpenSpec workspace
(`openspec context --json` resolves) in the project.

Produce a COMPLETE OpenSpec change — `proposal.md`, `specs/<cap>/spec.md`,
`design.md`, `tasks.md` — all validated and ready for `openspec-apply-change`.

Respond and write in the user's language.

---

## 0. Mode: new vs refine

- No argument or "new" → **new** change.
- `refine` → **refine**: run `openspec list --json`, show the top 3–4 active
  changes (name, schema, task progress, lastModified), let the user pick one.
  Edit its existing artifacts IN PLACE using paths from
  `openspec status --change <name> --json`. Never create a new change dir.
- A bare name argument → **new** with that proposed kebab-case name.

---

## 1. Baseline recon (parallel, before first question)

Dispatch both scouts in parallel and wait for both results before grilling:

- **openspec-scout** — existing specs, active changes, capability overlap,
  whether the proposed change name is free.
- **codegraph-scout** — entry points, call paths, blast radius, reusable
  patterns, risky couplings.

If codegraph-scout reports no index, tell the user and suggest `/codegraph-init`.
Never run any init yourself. Use the recon findings to eliminate questions you
could have answered yourself.

If your client cannot dispatch subagents (no openspec-scout / codegraph-scout
available), run the same reconnaissance yourself, read-only, before grilling.

---

## 2. Grill — Proposal decisions

Interview to pin down the four proposal sections. Each round:
- Present the FULL current frontier: questions whose prerequisites are already
  settled. Number each, give lettered options and a recommended default.
- Single focused question only for high-risk / irreversible decisions.
- Dispatch a scout mid-grill when a new factual gap appears. Do not block the
  whole frontier on it — ask the rest now, gate only downstream questions.
- Drive toward: Why (crisp), What Changes (scoped), Capabilities (every new/
  modified capability named), Impact (grounded in codegraph-scout blast radius).

Keep grilling until all four sections are fully resolved.

---

## 3. Create change directory (new mode only)

Once the kebab-case name is confirmed free by openspec-scout, run:
```
openspec new change "<name>"
```
Then get authoritative paths:
```
openspec status --change "<name>" --json
openspec instructions proposal --change "<name>" --json
```
Use `resolvedOutputPath` from instructions — never hardcode paths.

---

## 4. Write proposal.md

Write exactly these sections (from the schema template):
- `## Why` — problem/motivation, 1-2 sentences.
- `## What Changes` — concrete bullet list; mark breaking changes `**BREAKING**`.
- `## Capabilities` — `### New Capabilities` and `### Modified Capabilities`,
  each with a kebab-case capability path. Every change must declare at least one
  capability, or get explicit user confirmation before setting `skip_specs: true`.
- `## Impact` — affected code/specs/APIs, risks, migration, grounded in recon.

---

## 5. Parallel: proposal review + draft remaining artifacts

**Immediately after writing proposal.md, launch two tracks in parallel and wait
for both before proceeding.**

### Track A — Independent proposal review
Dispatch a fresh-context reviewer to audit the proposal against:
- User's stated intent from the grill session
- Recon findings (scope drift, missing capabilities, unchecked blast radius)
- Structural validity: are all four sections present and non-empty?
- Coherence: does Impact reflect what's in What Changes and Capabilities?

The reviewer must NOT edit any file. Return findings only.

### Track B — Draft specs, design, tasks (worker, using proposal as input)
While the reviewer audits, dispatch a worker to draft the three remaining
artifacts using the freshly written proposal.md as the sole source of truth.

The worker must:
1. Run `openspec instructions specs --change "<name>" --json` — write one delta
   spec per declared capability at `resolvedOutputPath`. For each capability:
   - `## Purpose` (new capabilities only)
   - `## ADDED / MODIFIED / REMOVED / RENAMED Requirements`
   - Each requirement: `### Requirement: <Name>`
   - Each scenario: `#### Scenario: <Name>` with `- **WHEN**` and `- **THEN**`
     (exactly 4 hashtags — 3 fails silently). Every requirement needs ≥1 scenario.
2. Run `openspec instructions design --change "<name>" --json` — check the
   `instruction` field. Write design.md only if the instruction says to (it is
   conditional in spec-driven). If skipping, record the reason and tell the main
   agent.
3. Run `openspec instructions tasks --change "<name>" --json` — write tasks.md
   as a flat `- [ ]` checklist ordered by dependency.

The worker derives all content from the proposal. Where the proposal is
ambiguous, make a reasonable assumption and flag it explicitly in the draft
for the main agent to resolve.

---

## 6. Reconcile review findings

Once both tracks finish:

1. Read the reviewer's findings.
2. For each material finding (scope gap, missing capability, contradicted impact):
   - Convert it into a grilling question.
   - Confirm the resolution WITH THE USER before editing anything.
3. Apply only the confirmed changes to proposal.md.
4. For each change to proposal.md, identify which downstream artifact(s) it
   affects and patch only those sections. Do not rewrite the whole artifact
   unless the proposal change was structural (capability added/removed).

---

## 7. Final validate + status check

```
openspec validate --change "<name>" --strict
openspec status --change "<name>" --json
```
Confirm all artifacts are `done` (or legitimately `skipped`).
Fix any validation errors before proceeding.

---

## 8. Deliver and hand off

Present the validated, complete change:
- Change name and location
- Artifacts created with a one-line summary each
- Any assumptions the worker flagged (user should verify these)
- Any conditional artifacts skipped and why
- `openspec validate --strict` result

Then stop and offer the handoff:
> "All planning artifacts are ready. Run `/openspec-apply-change` or ask me to
> start implementing when you're ready."

Do not start implementation in this workflow.

---

## Safety
- Two scouts are read-only. Never let them edit files.
- The reviewer in Track A is read-only. It audits, never edits.
- The worker in Track B writes only the three artifact files using paths from
  `openspec instructions --json`. It does not touch proposal.md.
- Never run `openspec init` or `codegraph init` automatically.
- Use `openspec status/instructions --json` for all paths — never hardcode.
- Treat the harmless `Rules for 'tasks'...` CLI warning as cosmetic.
- Never commit to git unless the user explicitly asks.
