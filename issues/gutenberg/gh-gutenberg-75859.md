# #75859: Block Supports: Add background gradient support that can combine with background images

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Blocks`, `[Package] Block library`, `[Package] Block editor`, `Needs Dev Note`, `[Block] Group`, `Global Styles`, `[Feature] Design Tools`, `[Package] Style Engine`
- **Merged:** [`c21d3ce`](https://github.com/WordPress/gutenberg/commit/c21d3ce958e9c4c6f2f5053fa995a06d9740721a)
- **Discussion:** [#75859](https://github.com/WordPress/gutenberg/pull/75859) · 28 comments · 2 reactions
- **Usefulness:** 4/5

## Summary

Introduces `background.gradient` as a new block support, enabling blocks to render gradients in the Background panel and combine them with background images via a single `background-image` CSS property. Previously, gradients were isolated to the Color panel as a standalone `background` shorthand that could conflict with or override explicit `background-image` declarations.

## Impact

['**Theme & plugin authors**: Opt-in blocks to `background.gradient` in `block.json` or `theme.json`; existing `color.gradient` remains functional and requires no immediate migration.', '**Editors & site builders**: Can now layer gradients over background images directly in the Block Editor’s Styles > Background panel.', '**Hosting & platform teams**: No breaking changes or configuration updates required; safe CSS sanitization is updated to permit combined gradient+URL values.']

## Technical details

The PR extends schema constants (`STYLE_PROPERTY`, `VALID_SETTINGS`, etc.) and the `theme.json`/`block.json` specifications to recognize `background.gradient`. The style engine (JS and PHP) is updated to read `style.background.gradient` and merge it with `backgroundImage` into a comma-separated `background-image` declaration. Frontend rendering in `blocks/background.php` handles the combined output, while `kses.php` registers a `safecss_filter_attr_allow_css` filter so core's `safecss_filter_attr()` does not strip gradient+URL combinations. Editor UI leverages `ColorPanelDropdown` to place the picker in the Background panel and conditionally suppresses the Color panel's gradient tab when enabled via `useSettingsForBlockElement`. The Group block is explicitly opted-in as the initial consumer.

## Contribution

PR #75859 was authored by @aaronrobertshaw and merged in commit c21d3ce. The implementation follows a strictly layered architecture across six commits covering schema updates, style engine logic (JS/PHP), backend rendering/KSES adjustments, editor UI components, Group block opt-in, and corresponding test suites. Review focused on verifying the separation of concerns; no alternative approaches were formally proposed or rejected in the available discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
