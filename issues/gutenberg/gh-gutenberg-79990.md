# #79990: Widget Dashboard: reserve paint space for tile focus rings

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] Widget Dashboard`
- **Merged:** [`bbd0c7a`](https://github.com/WordPress/gutenberg/commit/bbd0c7a4158de4a108884fdfd9514e141e243c46)
- **Discussion:** [#79990](https://github.com/WordPress/gutenberg/pull/79990) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Widget Dashboard package now reserves top padding in its grid container to prevent outward focus rings on widget tiles from being clipped by `overflow: auto` scroll containers. It also standardizes all CSS module class selectors to kebab-case and removes the package's stylelint suppressions, aligning internal naming conventions with project-wide linting rules.

## Impact

- **Dashboard & Widget developers**: No immediate code changes required for standard usage. External CSS that targets previous camelCase class names (e.g., `.widgetChrome`, `.tileEditMode`, `.editActionsEnter`) will need updating to their kebab-case equivalents.
- **Theme/Plugin authors extending the dashboard**: Custom overrides referencing internal component classes must switch to kebab-case selectors or use CSS variables (`--wp-widget-dashboard-tile-focus-offset`) for customizing focus ring spacing.
- **Site owners & editors**: No action required; keyboard navigation outlines are now fully visible at the top edge of scrollable widget grids without layout shifts.

## Technical details

The unified diff modifies multiple files within `packages/widget-dashboard/src/components/`:
- Adds CSS custom properties to the grid container in `widgets.module.css`: `--wp-widget-dashboard-tile-focus-offset: 2px;` and `--wp-widget-dashboard-focus-ring-reserve: calc(var(--wpds-border-width-focus) + var(--wp-widget-dashboard-tile-focus-offset))`, then applies `padding-block-start: var(--wp-widget-dashboard-focus-ring-reserve);` to reserve space for outward focus rings.
- Updates `.widget-chrome:has(:focus-visible)` and `.tile-edit-mode:focus-visible` to use `outline-offset: var(--wp-widget-dashboard-tile-focus-offset, 2px);` instead of hardcoded values.
- Renames all CSS module selectors to kebab-case (e.g., `widgetChrome` → `widget-chrome`, `tileEditMode` → `tile-edit-mode`, `editActionsEnter` → `edit-actions-enter`). Corresponding TSX files now reference these via bracket notation (`styles['kebab-case']`) to satisfy the `selector-class-pattern` lint rule.
- Removes all `packages/widget-dashboard` entries from `tools/stylelint/stylelint-suppressions.json`.

## Contribution

Opened and merged by @retrofox, with co-authorship noted as @simison. The change addresses a visual clipping issue where outward focus rings on top-row widget tiles were hidden by scroll container overflow, bringing upstream consistency with a similar fix applied in Automattic's Jetpack Premium Analytics package. Review remains lightweight, focusing on applying project-wide CSS naming conventions without altering component behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
