---
description: Set the technical HOW — stack, patterns, conventions, design
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
  - Bash
  - AskUserQuestion
---
You are setting the product's technical architecture.

Read: docs/prd.md, docs/stories.md
Output structure: @templates/architecture.md

Apply the codebase-analysis skill to analyze the starting code (boilerplate): structure, conventions, existing patterns. This is code the user didn't write — map it before deciding anything.

Proceed as follows:
1. The boilerplate question (AskUserQuestion): does the project start from a boilerplate? killer-saas is built to stand on one — the stack comes from the base, not from scratch.
   - Boilerplate present: analyze it (next steps).
   - No boilerplate: propose the options (AskUserQuestion) —
     a) **The Legion `nextjs-app` scaffold — the default.** Next.js SaaS Starter: Next.js 15 + React 19, Tailwind 4, shadcn/ui (Radix + CVA, `components.json`), Drizzle + PostgreSQL, Stripe subscriptions and Customer Portal, email/password auth with JWT in cookies, RBAC owner/member, activity log, route-protecting middleware. Package manager: **pnpm**. It lives on the `hermes` server and is NOT a git repo — copy it in, don't clone it:

            rsync -a --exclude node_modules --exclude .next \
              hermes@hermes:/home/hermes/.legion/scaffolds/nextjs-app/ ./
            pnpm install && cp .env.example .env

        Then rerun /ks-architect to analyze the result like any boilerplate. Sibling scaffolds sit next to it (`api-backend`, `expo-app`, `desktop-app-tauri`, `desktop-app-electron`): when the PRD isn't a web SaaS, name the right one instead of forcing this one. If `hermes` is unreachable, say so and offer option b rather than improvising a copy.
     b) Next.js + Tailwind + shadcn/ui scaffolded from scratch (+ Drizzle and auth if the PRD needs data/auth) — the fallback when the scaffold isn't reachable or doesn't fit. Same component vocabulary as the design system bound in /ks-design-system, but none of the SaaS plumbing: you rebuild auth, payments and the data layer by hand. Record the stack choice as an ADR, scaffold it (create-next-app, shadcn init), then analyze the result.
     c) Blank repo, assumed: record the chosen stack as ADRs instead of extracting conventions — and say it plainly: the method loses its main speed lever.
2. Analyze the existing repo and document its actual structure and conventions.
3. Fill the architecture template from this analysis + the PRD needs.
4. Check/complete the AGENTS.md file at the root with the concrete technical conventions ("Technical conventions" section).
5. Record each imposed structural decision (stack, patterns, integrations) as an ADR in `docs/decisions/NNN-<slug>.md`, following @templates/adr.md — with the considered options and why they were rejected.
6. Write the architecture to `docs/architecture.md` and commit it together with AGENTS.md and the ADRs on the default branch (docs: architecture).

End with: "Architecture ready + AGENTS.md updated. Next step: /ks-design-system (once), then /ks-research <story>"
