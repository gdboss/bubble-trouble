# Figma Design → EDS Block Variant Migration Plan

## Overview
Migrate an existing Figma design (provided as exported assets) into a **variant of an existing block** in the `bubble-trouble` project. The dedicated Figma-to-EDS tooling (`excat-figma`) will be enabled first, since you approved it, so design tokens and content can be extracted with proper support rather than by hand.

## Decisions Captured
- **Plugin:** Enable `excat-figma@excat-extended` (approved). You've confirmed to go ahead and enable it.
- **Source:** Exported Figma assets (images/specs), not a live Figma URL.
- **Target:** A new **variant of an existing block**.

## Status
- ⏳ **Plugin enablement is pending Execute mode.** Writing `.agents/settings.json` is a file modification, which plan mode does not permit. Switch to Execute mode and I'll write the file immediately (first action below), then proceed once it loads on the following message.

## Open Questions (to resolve before styling)
- **Which existing block** should get the new variant? Current blocks in this project: `cards`, `columns`, `footer`, `fragment`, `header`, `hero`. (The design's structure will confirm the best fit.)
- **Variant name** (e.g. `hero.event`, `cards.feature`).
- **Where the exported assets live** — the folder/path containing the Figma exports.

## Prerequisites
- `excat-figma` plugin enabled (see checklist).
- Exported Figma assets accessible and identified.
- Local preview available to visually verify the result.

## Checklist
- [ ] **Enable the `excat-figma@excat-extended` plugin** — write `.agents/settings.json` as `{"enabledPlugins": {"excat-figma@excat-extended": true}}` *(requires Execute mode; this is the first execution step)*
- [ ] After the plugin loads (next message), confirm its skills are available before proceeding
- [ ] Confirm which existing block the design maps to (`cards`, `columns`, `hero`, etc.) and choose a variant name
- [ ] Locate and inventory the exported Figma assets (images, spec/measurements)
- [ ] Extract design tokens (colors, typography, spacing) from the Figma exports via the Figma tooling
- [ ] Extract content/structure from the design and map it to the chosen block's content model
- [ ] Confirm the variant's content model works for authors (no breaking changes to the base block)
- [ ] Add the variant CSS to the block's stylesheet, extending — not overwriting — the base block
- [ ] Add any needed JS decoration for the variant (only if structure requires it)
- [ ] Add downloaded/optimized design images to the project
- [ ] Author a sample page/section using the new variant for verification
- [ ] Visually verify the variant in preview against the Figma design; iterate on spacing/color/type
- [ ] Run lint and block tests before finalizing

## Notes
- Enabling the plugin and all file changes above **require Execute mode** — plan mode is read-only, so I cannot write `.agents/settings.json` right now. Once you switch to Execute mode, the plugin file is written first; it loads on your following message, after which the Figma migration proceeds with the dedicated tooling.
- The plugin's skills won't be callable on the same turn the settings file is written — they become available on the next message after auto-reinitialization. No session restart is needed.
- I'll extend the existing block rather than modify its default behavior, so existing content using that block stays intact.
