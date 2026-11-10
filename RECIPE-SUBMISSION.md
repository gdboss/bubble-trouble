# AI Recipe Library submission (draft)

## Build Type
Workflow

## Title
Design token guardrails for Edge Delivery Services, enforced while Claude Code writes your CSS

## Overview
AI coding tools are fast at writing Edge Delivery Services block CSS, but they quietly break the design system: a hard-coded `#dadada` here, a made-up `var(--color-brand-accent)` there, and sometimes a `stylelint-disable` comment to make the error go away.

This recipe splits the work between AI and scripts:

- **Design tokens** live in one `tokens.json` file (W3C DTCG format). A small Node script writes them into `styles/styles.css` as CSS variables. There's no runtime build step, nothing extra ships to the browser, and authoring in Document Authoring / Experience Workspace is unchanged.
- **Quality gates** in `npm run verify` (Stylelint, ESLint, Node's built-in test runner) fail the build when anyone hard-codes a colour, uses a variable that doesn't exist, hand-edits the generated CSS, disables the colour lint rules, or picks token colours that fail WCAG AA contrast.
- **A Claude Code skill** (`/apply-design-tokens`) does the judgement work. It maps each hard-coded value to the right token by role, not by nearest hex. It loops until `npm run verify` passes, stops after 3 failed attempts, and writes a report listing every visual change and anything that needs a design decision.

The AI proposes; the scripts decide. The same gates run in GitHub Actions on every push, so they also protect against hand-written CSS.

## Goals + Results
Goal: let developers use Claude Code on EDS blocks without design-system drift reaching production.

Results on the public AEM boilerplate:
- 7 hard-coded colours (global styles + cards block) replaced with tokens in one run. `npm run verify` went from failing to passing.
- 46 automated tests; the full gate runs in about 4 seconds locally.
- 5 deliberate "cheats" (lint-disable comment, invented token, hand-edited generated CSS, low-contrast token, colour in `hsl()`) all caught by the gates.
- Zero runtime dependencies. The only thing that ships is about 25 extra lines of CSS variables in `styles.css`.

## Lessons learned
- **Don't let the AI grade its own work.** In my test, a `stylelint-disable-line` comment made the linter pass. Only a separate test that scans for colour-rule disables caught it. Guard the guardrails.
- **Hallucinated tokens are the most common AI failure.** `var(--color-brand-accent)` looks right and fails silently in the browser. A cross-file "is this variable defined anywhere?" check is cheap and catches it.
- **Map by role, not by nearest hex.** `#dadada` is a border, so it gets `--color-border-subtle`, even if a text grey is closer in value. Spell this out in the skill or the AI picks by colour distance.
- **Give the AI a way to stop.** "If no token fits, leave it and report it" produces better results than letting it invent tokens or edit `tokens.json`.
- **Keep legacy variable names** (`--link-color` etc.) and point them at tokens. Every existing block keeps working, and the diff stays small and reviewable.
- **Tighten one category at a time.** Start with colours. Spacing tokens are generated but not enforced yet, because block CSS uses px legitimately for borders and breakpoints.
- Passing gates prove consistency, not good looks. Changing token values changes the site, so always review the `aem.page` preview link.

## Instructions / Workflow
(Leave "Show Copy button" OFF; these are steps, not a single prompt.)

1. Clone the sample repo, or copy into your own EDS repo: `design-tokens/`, `tools/design-tokens/`, `test/design-tokens/`, `.claude/skills/apply-design-tokens/`, plus the `package.json` scripts, `.stylelintrc.json` rules, `.eslintrc.js` overrides and `.hlxignore` lines.
2. Add the two marker comments at the top of `:root` in `styles/styles.css` (see `design-tokens/README.md`).
3. Replace `design-tokens/tokens.json` and `contrast-pairs.json` with your design system's colours and the text/background pairs your site uses.
4. Run `npm install` then `npm run verify`. Expect it to fail and list every hard-coded colour. That's your baseline.
5. In Claude Code, from the repo root, run `/apply-design-tokens`.
6. Read the report: check "Visual changes for review" and "Needs a design decision".
7. Run `npx -y @adobe/aem-cli up` and check affected pages locally.
8. Open a pull request with the `{branch}--{repo}--{owner}.aem.page` preview link. GitHub Actions runs `npm run verify` on every push from now on.
9. Optional: try the 5 "prove the gates work" commands in the README to see each cheat get caught.

## Sample asset or link
https://github.com/<your-account>/eds-design-token-guardrails  ← replace once published

## Setup requirements
- An Edge Delivery Services site in GitHub (based on aem-boilerplate), with Document Authoring or any other content source
- Claude Code (Claude Pro, Max, Team or Enterprise, or an API key)
- Node.js 22 or later
- Your design system's colour values (a sample neutral token file is included)

## Tags
- Use case: Design system / Development (pick the closest option in the dropdown)
- Role / function: Developer (also relevant to front-end leads and design system owners)
- Adobe products used: Experience Manager
- AI platforms used: Claude
