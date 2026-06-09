# #78661: [Content Types]: Fix extra Page padding causing vertical scrollbar

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @yogeshbhutkar
- **Labels:** `[Type] Bug`, `Content types (experimental)`
- **Merged:** [`c17ad12`](https://github.com/WordPress/gutenberg/commit/c17ad12abb46b5b2a454a087437cb82ac6fc1302)
- **Discussion:** [#78661](https://github.com/WordPress/gutenberg/pull/78661) · 3 comments · 0 reactions

## Summary

The Settings → Content Types dashboard screen was displaying an unnecessary vertical scrollbar because padding was applied directly to the `.content-types-page` container. This change removes that padding from the page wrapper and applies it to the tab wrapper instead, eliminating the viewport overflow while aligning the layout with other experimental screens.

## Impact

- **Site owners & editors:** No functional changes; the dashboard renders cleanly without an unwanted scrollbar.
- **Plugin & theme developers:** No action required. The fix is scoped entirely to the experimental `content-types` package and does not alter public APIs, REST endpoints, or core editor behavior.
- **No breaking changes, deprecations, or migration steps.**

## Technical details

The change modifies `packages/content-types/src/components/layout/style.scss` to resolve a CSS box-model overflow.

Previously, `.content-types-page` applied a uniform padding that expanded the container height beyond the viewport. The diff removes that rule and shifts the spacing to `.content-types-tabs-wrapper`, applying contextual inline and block-start padding using design system variables (`--wpds-dimension-padding-2xl` and `--wpds-dimension-padding-lg`). This matches the spacing pattern used in related screens like Connectors and Experiments.

**Before:**
```scss
.content-types-page {
	 padding: var(--wpds-dimension-padding-lg);
}

.content-types-tabs-wrapper {
	 border-bottom: 1px solid $gray-100;
}
```

**After:**
```scss
/* .content-types-page rule removed entirely */

.content-types-tabs-wrapper {
	 padding-inline: var(--wpds-dimension-padding-2xl);
	 padding-block-start: var(--wpds-dimension-padding-lg);
	 border-bottom: 1px solid $gray-100;
}
```

## Contribution

Opened by @yogeshbhutkar to address a layout overflow observed during testing of the experimental Content Types feature. Review feedback from @im3dabasia suggested aligning the padding pattern with other screens (Connectors, Experiments), which was incorporated into the final diff. Co-authored by @ntsekouras and merged under commit `c17ad12`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
