# Sample run: `/apply-design-tokens` on the AEM boilerplate

Starting point: `adobe/aem-boilerplate` (commit `3acb6d9`), plus the tooling from this recipe.
Baseline `npm run verify`: **FAIL**, with 7 Stylelint colour errors.

## Design token report
Violations: 7 -> 0      npm run verify: PASS (46/46 tests, ~4 s)

| File:line | Before | After | Old value -> New value | Why this token |
|---|---|---|---|---|
| styles/styles.css:44 | `--background-color: white` | `var(--color-surface-default)` | #fff -> #fff | Page background |
| styles/styles.css:45 | `--light-color: #f8f8f8` | `var(--color-surface-subtle)` | #f8f8f8 -> #f5f6f8 | Used as section, footer and secondary button background |
| styles/styles.css:46 | `--dark-color: #505050` | `var(--color-text-muted)` | #505050 -> #565d6b | Used as text colour on light surfaces (secondary buttons) |
| styles/styles.css:47 | `--text-color: #131313` | `var(--color-text-default)` | #131313 -> #14171c | Body text |
| styles/styles.css:48 | `--link-color: #3b63fb` | `var(--color-text-link)` | #3b63fb -> #2f5bd3 | Links and primary button |
| styles/styles.css:49 | `--link-hover-color: #1d3ecf` | `var(--color-text-link-hover)` | #1d3ecf -> #1f3f9e | Link and primary button hover |
| blocks/cards/cards.css:11 | `border: 1px solid #dadada` | `var(--color-border-subtle)` | #dadada -> #d9dce1 | Decorative card border |

### Visual changes for review
All except `--background-color` change slightly. The biggest is the link/primary button blue (#3b63fb -> #2f5bd3, darker). Check buttons, links, cards and the footer on the preview link.

### Needs a design decision
None.

### Contrast pairs to add
None. Every remapped legacy pair is in `contrast-pairs.json`. Not testable: hero text (`--background-color`) over the hero image. Check by eye.

## Gate check: deliberate breakage

Each change below was applied on its own to the passing branch. `npm run verify` failed every time:

| Attempt | Caught by |
|---|---|
| `#dadada` + `stylelint-disable-line color-no-hex` in cards.css | Repo test: no colour-rule disables |
| `var(--color-border-strong)` (doesn't exist) | Repo test: undefined variables |
| Hand-edited `--color-brand-500` in the generated block | `tokens:check` |
| `text.muted` changed to #9aa0aa | Contrast test: 2.63 < 4.5 |
| `hsl(0deg 0% 85%)` in cards.css | Stylelint `function-disallowed-list` |
