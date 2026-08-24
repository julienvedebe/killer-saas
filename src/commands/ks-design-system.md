---
description: Bind the project to an Open Design design system and mirror it into docs/design-system.md
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Bash
  - AskUserQuestion
  - ListMcpResourcesTool
  - ReadMcpResourceTool
  - mcp__open-design__list_projects
  - mcp__open-design__create_project
  - mcp__open-design__get_project
  - mcp__open-design__start_run
  - mcp__open-design__get_run
---
# ks-design-system — Global design system, bound to Open Design

This command does NOT draw. It picks — or commissions — the product's design system in Open Design, binds it to a project, and mirrors it into `docs/design-system.md`: the versioned doc that every later phase reads, from /ks-design to the reviewer.

## Execution contract (non-negotiable)
You are FORBIDDEN from:
- Inventing a visual identity from nothing, or writing a generic default design system (random colors, imaginary components) into docs/design-system.md.
- Writing docs/design-system.md without its Open Design binding: the file and the binding are one deliverable, and /ks-design fail-closes on the binding.
- Copying the target SaaS's visual identity. The target is a spec for structure and flows — never a brand you don't own.

## Workflow

### Step 0 — Open Design reachable (fail-closed)
Call `list_projects`. Error → STOP: "Open Design unreachable. Start it (or connect its MCP server), then rerun /ks-design-system." The whole design phase runs on it; there is no degraded path.

### Step 1 — Inventory the boilerplate
Apply the codebase-analysis skill to the starting code: which components ACTUALLY exist (real names, real import paths), and which tokens or theme the code already carries. This is the half Open Design cannot know — a design system naming components the repo doesn't have produces screens nobody can build.

Also determine whether the boilerplate has a **component registry** — a source it can pull more components from (a `components.json` means shadcn/ui, for instance). Record its name, its add command and where its catalog lives. This is what later separates a component that is merely *not installed yet* from one that *does not exist anywhere*: without it, every missing primitive escalates to this command, and nobody will keep doing that by the third form. No registry → say so by omitting the block, and accept the consequence: every missing component is then a gap.

### Step 2 — Choose the design system (AskUserQuestion)
List what's available first: the `od://design-systems/…` MCP resources — the catalog (shadcn, linear-app, stripe, material, minimal…) and the user's own `user:<slug>` systems. Then ask which route:

- **An existing system** — from the catalog, or one of the user's `user:*` systems.
- **Extract one from a live site** — `start_run` with skill `brand-extract` on a URL the user owns (their product, their company). Never the target SaaS's site: replicating a product is the method, copying its brand is not.
- **Author one** — `start_run` with skill `design-md` (or `design-consultation` for a from-scratch direction), briefed from the PRD's angle and the boilerplate's constraints.

The last two produce a `user:<slug>` system: poll `get_run` until terminal before going further, and stop on failure rather than continuing with no system.

Prefer the system that matches the boilerplate. The boilerplate is imposed (AGENTS.md): a design system fighting it makes every story pay a translation.

### Step 3 — Create and bind the product's project
`create_project({ name: "<product> — killer-saas", designSystem: <ds id>, skill: "frontend-design" })`.

**One Open Design project for the whole product.** Every story's screen lands in it, so the screens stay coherent with each other and not merely with the tokens — the drift a per-story project cannot see. If the project already exists (`list_projects`), reuse it; never create a second one for the same product.

### Step 4 — Mirror into docs/design-system.md
Read the chosen system's `od://design-systems/<ds id>/DESIGN.md` and write docs/design-system.md (structure: @templates/design-system.md):

1. **Frontmatter — the binding, and the registry.** Open Design project id, design system id, skill (plus agent/model only if the user pinned them). This is the pipeline's only pointer to Open Design: /ks-design reads it and refuses to run without it. Add the `component-registry` block from Step 1 — or omit it deliberately when there is none.
2. **The mirrored sections**, verbatim from DESIGN.md — direction, palette, typography, layout and posture rules. Verbatim, not summarized: the mirror is what /ks-plan and the reviewer read, and a summary is exactly where a screen and its system start to disagree.
3. **The boilerplate's real components** from Step 1 — name, import path, what the system's vocabulary calls it. Installed ones only: this table is what a story can use today. It is the translation layer /ks-plan and the implementer build from.
4. **Do / Don't**, merging the system's rules with the boilerplate's constraints.

The mirror is a copy, not a source. Change the system in Open Design, then rerun /ks-design-system to refresh it — never hand-edit a mirrored section, it will be silently overwritten and, worse, believed in the meantime.

### Step 5 — Commit
Commit docs/design-system.md on the default branch (docs: design system). Like AGENTS.md and the ADRs, it is a transverse asset: set once, read at every story.

### Step 6 — Settle the open gaps
If earlier stories recorded "Design system gaps" in their `docs/designs/<id>.md`, this is where they get settled — that is what sending them here was for. For each one: extend the design system in Open Design, adopt a registry component that covers it, or decide explicitly that the product does without. Record the decision; a gap that is merely re-read is a gap nobody settled.

End with: "Design system bound (<ds id>) and mirrored in docs/design-system.md. Open Design project: <project id>. Story screens: /ks-design <story>."
