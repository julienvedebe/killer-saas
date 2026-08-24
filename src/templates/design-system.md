---
open-design:
  project: <od project id>            # created by /ks-design-system, used by every /ks-design run
  design-system: <od design system id>  # catalog id (shadcn, linear-app…) or user:<slug>
  skill: frontend-design              # skill driving each story's run
  # agent: <runtime id>               # optional — only if pinned; default: Open Design's configured runtime
  # model: <model id>                 # optional — same
component-registry:                   # the boilerplate's own component source, if it has one.
  name: <e.g. shadcn/ui>              # OMIT THIS WHOLE BLOCK when there is none: every
  add: <e.g. pnpm dlx shadcn@latest add <component>>   # missing component is then a gap,
  catalog: <URL or command listing what the registry offers>   # settled in /ks-design-system.
---
# Design System — <product>

> Sections marked *(mirror)* are copied verbatim from the Open Design system named above.
> Never hand-edit them: change the system in Open Design, then rerun /ks-design-system.
> Everything else is this repo's own layer — the boilerplate reality Open Design can't see.

## Direction *(mirror)*
<the system's positioning and voice, as DESIGN.md states it>

## Tokens *(mirror)*
- Colors: <role | name | hex | usage>
- Typography: <display / body — families, weights, fallbacks, scale>
- Spacing / radius / borders: <...>

## Layout & posture rules *(mirror)*
- <the system's rules, verbatim — density, elevation, motion, iconography, anti-patterns>

## Available components (boilerplate)
Installed right now — what a story can use today without adding anything.

| Component | Import | Usage | Maps to (system) |
|---|---|---|---|
| <Button> | <@/components/ui/button> | <...> | <primary action> |
| <Input>  | <@/components/ui/input>  | <...> | <field> |

Anything the registry above offers but the repo hasn't installed is **installable**, not missing:
a story adds it with the registry's command, as a plan task. Anything no registry offers is a
**gap**, settled here and nowhere else.

## UI patterns
- Forms: <...>
- States (empty / loading / error / success): <...>
- Feedback (toast, inline): <...>

## Do / Don't
- ✅ <system rule + boilerplate constraint>
- ❌ <...>

<< IP Mike: the real system — which catalog system fits which boilerplate, the component mapping that actually holds. >>
