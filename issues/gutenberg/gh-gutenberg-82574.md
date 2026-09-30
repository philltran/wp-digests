# #82574: DOM: Exclude visibility-hidden elements from focusable results

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] DOM`, `[Package] Block editor`
- **Merged:** [`b79ddae`](https://github.com/WordPress/gutenberg/commit/b79ddae151853f810070501030e4265abd16c67d)
- **Discussion:** [#82574](https://github.com/WordPress/gutenberg/pull/82574) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`focusable.find()` and `tabbable.find()` in `@wordpress/dom` now exclude elements hidden by CSS `visibility: hidden`, `visibility: collapse`, or `content-visibility: hidden`. Previously such elements kept a layout box, so they passed the existing size check and were returned even though native `.focus()` cannot focus them. Descendants that explicitly restore `visibility: visible` inside a hidden ancestor are still returned. A companion guard in `NavigableToolbar` stops hidden custom block toolbars from remounting.

## Impact

- **Plugin & theme developers:** Code that calls `wp.dom.focus.focusable.find()` or `wp.dom.focus.tabbable.find()` will no longer receive elements hidden via `visibility` or `content-visibility`. Focus traps, roving-tabindex logic, or custom keyboard navigation that relied on those elements appearing in results (for example, to redirect focus) may behave differently.
- **Block authors with custom block toolbars:** Toolbars built from non-`ToolbarItem` controls are no longer reclassified and remounted while the toolbar is visibility-hidden and its subtree changes asynchronously.
- **Site owners / hosting / REST consumers:** No action required.
- No API signatures, hooks, or deprecations change.

## Technical details

**`packages/dom/src/focusable.js` — `isVisible()`**

- If `element.checkVisibility` exists, it calls `element.checkVisibility( { visibilityProperty: true } )` and returns `false` when that fails. This covers inherited `visibility` and `content-visibility: hidden` ancestors, while correctly honoring a descendant's `visibility: visible`.
- Otherwise it falls back to `element.ownerDocument.defaultView?.getComputedStyle( element ).visibility` and rejects `hidden` or `collapse`. Using the owning window makes this correct for elements inside iframes.
- The pre-existing layout-box check (`offsetWidth`/`offsetHeight`/`getClientRects()`) still runs afterward.

**`packages/block-editor/src/components/navigable-toolbar/index.jsx`**

`useIsAccessibleToolbar`'s `determineIsAccessibleToolbar` now returns early when `focus.tabbable.find( toolbarRef.current )` is empty. Previously an empty list made `hasOnlyToolbarItem( tabbables )` evaluate as true-ish for "every control is a `ToolbarItem`", so a hidden custom toolbar's classification flipped and the toolbar remounted after a subtree mutation.

```js
const tabbables = focus.tabbable.find( toolbarRef.current );
// A hidden toolbar has no tabbables, so keep its current classification.
if ( tabbables.length === 0 ) {
	return;
}
```

**Tests / tooling**

- New Chromium browser tests: `packages/dom/src/test/focusable.browser.test.js` and `navigable-toolbar/test/index.browser.test.js`.
- The JSDOM `createElement` test util no longer treats `visibility: hidden` as removing layout, and the JSDOM fixtures are now attached to `document.body` so computed styles invalidate.
- `test/unit/scripts/validate-test-routing.mjs` strips Vitest's trailing instance label (e.g. `browser (chromium)`) from project names.

Bundle impact: `dom/index.min.js` +57 B, `block-editor/index.min.js` +15 B.

## Contribution

Opened by @ciampo as a standalone extraction of the production fix that also lives in the #80995 migration stack, which will need deduplication after this merges. @ciampo highlighted it as an example of Browser Mode tests surfacing two subtle bugs (the `visibility` handling and the toolbar consumer), and @aduth noted that similar concerns about test mocking obscuring real behavior, and precedent for this solution, had come up in #79295. The PR states that Codex implemented and verified the change, including an independent code review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
