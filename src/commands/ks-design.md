---
description: Derive a story's screen from the design system — generated in Open Design, never drawn by hand.
argument-hint: <story id or name> [--refine "<feedback>"]
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - AskUserQuestion
  - mcp__open-design__get_project
  - mcp__open-design__start_run
  - mcp__open-design__get_run
  - mcp__open-design__get_artifact
  - mcp__open-design__list_files
---
# ks-design — Story design, generated in Open Design

Target story: $ARGUMENTS

## Execution contract (non-negotiable)
You are FORBIDDEN from:
- Producing a design without an existing design system (see Step 1).
- Drawing the mockup yourself. Open Design generates the screen; you commission it, poll it, and bring the result back. There is no fallback path — Open Design unreachable means the step stops, it does not degrade into an agent-drawn mockup.
- Inventing a component, token, color or spacing outside the design system.
- Designing a screen the story doesn't ask for.
- Writing docs/designs/<id>.html from a run that did not succeed, or editing that file by hand afterwards.

## Workflow

### Step 1 — Prerequisites (fail-closed)
1. `docs/design-system.md` exists AND is non-empty. Missing or empty → STOP. Reply: "No design system found in docs/design-system.md. Set it up first via /ks-design-system, then rerun /ks-design." Produce NO design.
2. Its frontmatter carries the Open Design binding:

       open-design:
         project: <od project id>
         design-system: <od design system id>
         skill: <od skill id>

   Missing or incomplete → STOP: "docs/design-system.md carries no Open Design binding. Rerun /ks-design-system to bind the project." Never invent a project id, and never fall back to Open Design's active project: the binding in the file is the state, the open tab is not.
3. Resolve the project: `get_project(project: <od project id>)`. Error, or a project whose `designSystemId` no longer matches the binding → STOP and say exactly what failed (Open Design down, project deleted, system detached). Do not create a project here — `/ks-design-system` owns that.

### Step 2 — Read the story
Read docs/stories.md, resolve the target story id (`s<number>-<slug>`) and isolate its acceptance criteria. Read docs/research/<id>.md if it exists — its anchor points tell you which pages and layouts the screen plugs into. If the PRD names a target SaaS, its equivalent screen is a layout/UX reference — structure, fields and states only, never visual identity: the identity comes from the bound design system and from nowhere else. The design covers this screen only.

Stories without UI skip this step: say so and point to /ks-plan.

### Step 3 — Write the brief (the brief IS the prompt)
Write docs/designs/<id>-brief.md (structure: @templates/design-brief.md). This file is the prompt sent to Open Design, recorded in the repo so the run is auditable and reproducible — it is a deliverable of this step, not a chat message.

Do NOT copy the design system's tokens into it. The system is bound to the Open Design project and applies to every run; recopying tokens by hand is how a screen drifts from the system it claims to follow. The brief states what Open Design cannot know: the story, the screen's purpose, the exact fields, labels and data shown, the actions, the four states (empty / loading / error / success), what is explicitly out of scope, and the output contract below.

### Step 4 — Commission the run
Call `start_run`:
- `project`: the bound project id.
- `skill`: the binding's `open-design.skill` (default `frontend-design`).
- `agent` / `model`: only if the binding names them. Otherwise let Open Design use its configured runtime.
- `requestId`: `ks-<id>` on the first run, `ks-<id>-r<n>` on refinement n. Generate it BEFORE the call and reuse it verbatim if the call has to be retried — a retry under the same id is the same run, not a second one.
- `prompt`: the brief's content, plus the output contract — produce ONE self-contained HTML file named `<id>.html` at the project root: inline CSS and JS, no external asset, no build step. Low fidelity is fine and expected: the mockup communicates layout and states, not production code.

Then poll `get_run(runId)` until the status is terminal.
- `succeeded` **and** `deliverableValid: true` → keep `runId`, `previewUrl` (and `studioUrl` when present), continue.
- `succeeded` with `deliverableValid: false` → treat it as a failure: the agent exited clean but the deliverable didn't validate. Report `deliverableValidation` and stop. An exit code of 0 is not a design.
- `failed` / `canceled` → STOP. Report the error and the runId. Write NO design file: a half-run is not a design.
- Paused for credit → STOP and say so. Topping up is the user's decision; resuming means rerunning /ks-design with the same requestId.
- A run takes minutes, not seconds. Poll it. Never start a second run because the first is slow — that bills twice for one screen.
- Files can appear in the project BEFORE the run reaches a terminal status — the inner agent writes, then keeps polishing. Poll the status, never the file listing: a file that exists is not a file that is finished.
- If the run returns an `agentMessage` and no preview, the design agent asked a question instead of producing files: relay the question, answer it in the brief, and rerun. Don't answer in its place.

### Step 5 — Bring the result back
1. `get_artifact(project: <od project id>, entry: "<id>.html")`.
2. Write the entry file verbatim to docs/designs/<id>.html. Verbatim: you are the transport, not a co-author.
3. If the run produced sibling files despite the single-file contract, write them under docs/designs/<id>.assets/ and record it in the .md — a mockup nobody can open from the PR has stopped being a reference.
4. A wrong result is refined (Step 7), never patched by hand: a mockup you edited is no longer the screen the design system produced, and the next run overwrites your edit anyway.

### Step 6 — Record the screen
Write docs/designs/<id>.md (structure: @templates/design-screen.md), including the run's traceability block: Open Design project, design system id, run id, preview URL. The components section names the design system's components the screen reuses — the reviewer checks the built screen against them.

The run's final message is part of the deliverable: it names the gaps it hit and the fixes it made to itself. Read it and carry its gaps into the .md — they are the ones the system actually failed to cover, observed rather than guessed.

Gaps: any need the design system doesn't cover → record it under "Design system gaps" in the .md. DON'T invent it, and don't let the run invent it either: a gap is settled in /ks-design-system, not here.

### Step 7 — Refine
`/ks-design <id> --refine "<feedback>"`: same project, same design system, `start_run` with the feedback plus a pointer to the existing `<id>.html`, `requestId` `ks-<id>-r<n>`. Then redo Steps 5 and 6. Append the refinement to the brief so that file keeps the full intent — the brief is the prompt's history, not just its first draft.

Timebox: defined enough to unblock the Plan, not pixel-perfect. Two refinements is the normal ceiling; past that, the problem is the story or the design system, not the prompt.

## Mockup status (hard rule)
docs/designs/<id>.html is a REFERENCE, not code to copy. In Execute, the screen is built with the boilerplate's real components. The mockup communicates intent (layout, states); it doesn't replace the component system and never gets pasted into production.

End with: "Design ready (docs/designs/<id>.md + .html — Open Design run <runId>). Next step: /ks-plan <id>"
