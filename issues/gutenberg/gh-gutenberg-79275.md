# #79275: DataForm: Fix panel field control overflow clipping and remove button overrides

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`1120cf2`](https://github.com/WordPress/gutenberg/commit/1120cf2cd3675ae62f8c9dd17f0ef223cb126633)
- **Discussion:** [#79275](https://github.com/WordPress/gutenberg/pull/79275) · 6 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The DataForm panel layout in the DataViews package now uses `overflow: clip` instead of `hidden` on field controls, preventing inner element focus rings from being clipped. The change also removes legacy CSS overrides for `.components-button` and `.components-dropdown` that were artifacts from earlier panel refactorings. This improves keyboard navigation visibility and cleans up internal styling without altering public APIs.

## Impact

- **DataViews & Site Editor consumers:** Focus rings on interactive controls inside DataForm panels will render fully, improving accessibility for keyboard users.
- **Plugin & theme developers:** No breaking changes. The removed CSS overrides were internal to the DataViews package and targeted components rendered inside panel field controls. External consumers relying on these specific overrides would need to adjust, but the PR notes the impact is minimal.
- **No action required** for standard upgrades.

## Technical details

The change modifies `packages/dataviews/src/components/dataform-layouts/panel/style.scss`. The `.dataforms-layouts-panel__field-control` rule switches from `overflow: hidden` to `overflow: clip` and adds compensating `padding-inline` and `margin-inline` values (`var(--wpds-border-width-focus)`) to ensure focus rings remain visible within the clip boundary. It also strips out three legacy selector blocks that previously forced layout overrides on inner components.

**Before:**
```css
.dataforms-layouts-panel__field-control {
    overflow: hidden;
    /* ... */
    .components-button { max-width: 100%; text-align: left; white-space: normal; text-wrap: balance; text-wrap: pretty; min-height: vars.$button-size-compact; }
    &.components-button.is-link[aria-disabled="true"] { text-decoration: none; }
    .components-dropdown { max-width: 100%; }
}
```

**After:**
```css
.dataforms-layouts-panel__field-control {
    overflow: clip;
    padding-inline: var(--wpds-border-width-focus);
    margin-inline: calc(-1 * var(--wpds-border-width-focus));
    /* ... */
}
```

## Contribution

The changes were extracted from a larger PR (#79195) to keep the scope focused on the overflow fix and CSS cleanup. During review, @ciampo raised concerns about the underlying `SummaryButton` implementation and keyboard accessibility, but @ntsekouras opened a separate tracking issue (#79427) to address that separately, allowing this PR to ship the layout fix immediately.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
