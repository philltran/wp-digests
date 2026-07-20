# #80300: Notes: Finish WPDS treatment for mention chips

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Notes`
- **Merged:** [`c7b14b6`](https://github.com/WordPress/gutenberg/commit/c7b14b6cab32a2f80bd9a72ec809d4700b81ee6d)
- **Discussion:** [#80300](https://github.com/WordPress/gutenberg/pull/80300) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `.wp-note-mention` chip in the Notes sidebar now uses semantic WPDS design tokens instead of legacy Sass variables and hardcoded color mixes. This update adds proper interactive states (`:hover`, `:active`, `:focus`) and a `forced-colors` fallback to ensure the underline-less chip remains accessible and visually consistent across themes and high-contrast modes.

## Impact

- **Plugin & theme developers:** No action required. The change is scoped to the editor's internal Notes sidebar styling and does not expose public APIs, block filters, or theme hooks.
- **Hosting & platform teams:** No action required. The CSS diff is minimal (+531 B compressed) and relies on existing WPDS token fallbacks that track `--wp-admin-theme-color`.
- **Site owners:** No action required. The visual behavior improves without altering the Notes feature's functionality, data structure, or user workflow.

## Technical details

The diff modifies `packages/editor/src/components/collab-sidebar/style.scss` to replace legacy styling on `.wp-note-mention`:
- Swaps `$grid-unit-05` / `$radius-small` for `--wpds-dimension-padding-xs` / `--wpds-border-radius-sm`.
- Replaces `color-mix(in srgb, var(--wp-admin-theme-color) 12%, transparent)` and `var(--wp-admin-theme-color)` with `--wpds-color-background-surface-brand` and `--wpds-color-foreground-interactive-brand`.
- Adds explicit `:focus`, `:hover`, and `:active` rules using `--wpds-border-width-focus`, `--wpds-color-stroke-focus`, and `--wpds-color-background-interactive-brand-weak-active` / `--wpds-color-foreground-interactive-brand-active`.
- Includes a `@media (forced-colors: active)` block that restores `text-decoration: underline` when background tints are stripped.
- Clears inherited `wp-admin/common.css` focus styles by setting `box-shadow: none` and explicit `outline`/`border-radius` values.

## Contribution

Opened by @Mamaduka as a follow-up to PR #79604, the change was reviewed and merged by @ciampo, @adamsilverstein, and @jasmussen. During review, the team discussed whether the mention chip should be generalized into a reusable system component or shared across the editor and post content. The approach was considered but not included; the consensus was that while the styling aligns with the `Link` component, the chip remains an editor-specific UI element rather than a content block, so it stays scoped to the Notes sidebar without extracting a new public component. The PR was cherry-picked to `wp/7.1` for inclusion in the next release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
