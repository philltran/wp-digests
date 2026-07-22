# #80315: DataViews/Font Library: Give search fields a fixed width to stop layout shift

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Enhancement`, `[Package] DataViews`
- **Merged:** [`25d5c72`](https://github.com/WordPress/gutenberg/commit/25d5c72ceb0f452ce7829b2f1347c156b0d9a196)
- **Discussion:** [#80315](https://github.com/WordPress/gutenberg/pull/80315) · 13 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request resolves a layout shift in the `SearchControl` component by applying fixed widths to its consumers in the DataViews package and the Font Library. Previously, the search input visibly resized when the reset button mounted or unmounted, creating an unpolished experience in the Site Editor and Global Styles UI. The fix shifts the responsibility from the component internals to the consuming layouts, ensuring stable dimensions during user interaction.

## Impact

- **Plugin & theme developers:** No breaking changes or deprecations. If you wrap `SearchControl` in a flex container and relied on its previous `fit-content` behavior, you may need to adjust your container's flex properties to maintain the intended layout.
- **Site editors & core UI users:** The "Install fonts" screen and DataViews search fields (e.g., Pages, Templates, Patterns) will no longer jump in width when typing or clearing search values.
- **No action required** for standard implementations, as the change is scoped to core editor packages and does not alter public component APIs.

## Technical details

The diff applies CSS constraints directly to the two identified consumers rather than modifying `SearchControl` internals. In `packages/dataviews/src/components/dataviews-filters/style.scss`, the `.dataviews-search` selector is updated from `width: fit-content;` to `flex-basis: 210px;`, with a corresponding `flex-grow: 1;` added to `.dataviews__search` in `packages/dataviews/src/dataviews/style.scss` to preserve flex distribution. For the Font Library, `packages/global-styles-ui/src/font-library/font-collection.tsx` injects `className="font-library__search"` into the `<SearchControl>` component, and `packages/global-styles-ui/src/font-library/style.scss` defines `.font-library__search { width: 240px; }`.

Before/after pattern for DataViews filters:
```css
/* Before */
.dataviews-search {
	width: fit-content;
}

/* After */
.dataviews-search {
	flex-basis: 210px;
}
```
This approach avoids unmounting/mounting side effects and keeps the reset button's space reserved without altering the component's internal rendering logic.

## Contribution

Opened by @juanfra, the PR initially proposed a `SearchControl`-level fix that would always mount the reset button suffix and toggle its visibility instead of unmounting it. During review, @mirka suggested addressing the layout shift at the consumer level to avoid added component complexity and potential placeholder collision issues. @juanfra adapted the approach, applying explicit fixed widths to the DataViews and Font Library consumers instead. The change was reviewed by @ciampo and @fcoveram, and merged with co-authors @aduth, @mirka, @ciampo, and @fcoveram.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
