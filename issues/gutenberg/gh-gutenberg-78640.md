# #78640: List View: Remove redundant visibilityLabel compoutation

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`
- **Merged:** [`a009f9d`](https://github.com/WordPress/gutenberg/commit/a009f9dc63347eefe362cabec0ba6bd0cf7f309a)
- **Discussion:** [#78640](https://github.com/WordPress/gutenberg/pull/78640) · 2 comments · 0 reactions

## Summary

A redundant `getBlockVisibilityLabel` call in `ListViewBlockSelectButton` has been removed. The label was already computed by the parent `ListViewBlock` component and surfaced to assistive technology via `AriaReferencedText` + `aria-describedby` — the duplicated `useSelect` selector in the child component was unnecessary. The PR also corrects a misleading `TODO` comment introduced in #78411 that incorrectly described the label as sighted-hover-only.

## Impact

**Plugin & theme developers** — No action required. This is an internal refactor with no API surface changes.

**Hosting & platform** — No action required. No behavioral change to the editor, accessibility output, or block serialization.

**Headless & REST consumers** — Unaffected.

## Technical details

The change spans two files in `packages/block-editor/src/components/list-view/`:

- **`block.js` (`ListViewBlock`)** — already called `getBlockVisibilityLabel(block?.attributes?.metadata?.blockVisibility)` and placed the result into an `AriaReferencedText` node keyed to `descriptionId`. The row's `<a>` element references that node via `aria-describedby={ descriptionId }`, so screen readers already announce the visibility description as part of the row's accessible description. The label is now also forwarded to the child as a `visibilityLabel` prop.

- **`block-select-button.js` (`ListViewBlockSelectButton`)** — previously re-derived the same label with its own `useSelect` call and a second invocation of `getBlockVisibilityLabel`. Both are now dropped; the component accepts `visibilityLabel` as a prop instead. The removed `TODO` comment ("visibilityLabel is sighted-hover-only") is replaced with a comment pointing to the parent's `AriaReferencedText` to clarify the actual AT exposure path.

The net effect is one fewer store selector subscription per List View row that has block visibility metadata set. The minified bundle shrinks by 7 B (`build/scripts/block-editor/index.min.js`).

No hooks, filters, REST schema, or `block.json` fields were added or changed. No deprecations.

## Contribution

Opened and merged by @ciampo (Marco Ciampini); @mirka is credited as co-author via the props attribution. The PR directly resolves a `TODO` left in the codebase by #78411. The PR description notes it was authored with Cursor (Claude) assistance and reviewed by a human before commit. The discussion was minimal — two bot comments — with no recorded design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
