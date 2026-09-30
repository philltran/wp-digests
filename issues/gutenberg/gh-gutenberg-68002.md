# #68002: List: Add Align Support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shail-mehta
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] List`, `[Feature] Layout`
- **Merged:** [`6eddc65`](https://github.com/WordPress/gutenberg/commit/6eddc656ddd6018c0ee65aca698f46b3b59aad2a)
- **Discussion:** [#68002](https://github.com/WordPress/gutenberg/pull/68002) · 17 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The core `List` block (`core/list`) now declares `align` support for `wide` and `full`, so the alignment control appears in the block toolbar and the block can break out of the content width in themes that support wide alignment. The change is a single `supports` entry in `block.json`, plus docs and changelog updates. It is part of the broader effort to add missing block supports (tracking issue #43248).

## Impact

**Site owners / editors**
- The List block toolbar gains an alignment control offering Wide width and Full width in themes with `align-wide` support (and in block themes where layout permits it).
- Existing lists are unaffected until an alignment is explicitly chosen.

**Theme & plugin developers**
- No action required, no deprecations or removed APIs.
- Front-end output for an aligned list gets the `alignwide` / `alignfull` class on the `<ul>`/`<ol>` wrapper. Themes and stylesheets that target these classes generally, or that style `.wp-block-list` directly, may want to check how a wide or full list renders. Text is not automatically re-aligned inside a full-width list, so a list may look different from a constrained one.
- `core/list` block-support introspection (e.g. code reading `getBlockType('core/list').supports.align`) now returns `[ "wide", "full" ]` instead of being undefined.

**Headless / REST consumers**
- The block type's `supports` in the block-types REST response now includes `align: ["wide", "full"]`.

**Caveat noted in review**
- A reviewer observed that when the block width is not adjusted, the justification option does not visibly move the list items, which may make the setting look broken. The discussion in the record does not show how or whether this was resolved before merge.

## Technical details

The functional change is one line in `packages/block-library/src/list/block.json`:

```diff
 "supports": {
 	"anchor": true,
+	"align": [ "wide", "full" ],
 	"html": false,
```

Because `align` is a standard block support, the existing align support hook adds the `align` attribute and toolbar control and applies the `alignwide`/`alignfull` class. No JS or PHP changes to the List block were needed, and no new hooks or filters are introduced.

Other files in the diff:
- `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/list/README.md`: generated documentation now lists `align (full, wide)` under the block's supports.
- `packages/block-library/CHANGELOG.md`: adds an "Enhancements" entry, `List: Add wide and full alignment support`.
- `test/e2e/specs/editor/various/splitting-merging.spec.js`: an unrelated e2e tweak changing `.click()` to `.focus()` on the empty-block locator, presumably to stabilise the test. The record does not state why.

No saved-content migration or deprecation is needed: the addition is an opt-in attribute, and the block's saved markup is unchanged for lists without alignment.

## Contribution

The PR went through several iterations on the author's side: switching from `__experimentalLayout` to `layout` and rewriting the testing instructions after @aaronrobertshaw asked for clearer, actionable steps and a better title (he noted he lacked bandwidth to review). @carolinan later raised a UX concern that the justification option has no visible effect when the block width isn't adjusted. The PR credits list also includes @tellthemachines and @andrewserong.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
