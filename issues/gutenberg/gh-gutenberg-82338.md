# #82338: Icons: Convert Search icon to strokes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Package] Icons`
- **Merged:** [`0b466a1`](https://github.com/WordPress/gutenberg/commit/0b466a172502fa0d103ad4102dd5f24e22175049)
- **Discussion:** [#82338](https://github.com/WordPress/gutenberg/pull/82338) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The public `search` icon exported from `@wordpress/icons` is now a stroke-based glyph (`stroke="currentColor"`, `style="fill: none"`, non-scaling stroke) instead of a filled path. Because published themes style the Search block's front-end SVG with `fill`, the Search block editor now uses its own block-owned copy of the legacy filled icon, so the editor still matches the unchanged PHP-rendered markup.

## Impact

- **Plugin & theme developers using `@wordpress/icons`:** This is listed as a **Breaking Change** in the icons CHANGELOG. If you render the `search` icon and recolor it with CSS `fill`, that no longer works. Use CSS `color` instead. To deliberately override the intrinsic `fill: none`, pass `style={ { fill: value } }`.
- **Visual change:** The `search` icon now has a different (outlined) design wherever it is consumed from `@wordpress/icons`. Its stroke width stays constant at sizes other than 24px because of `vector-effect="non-scaling-stroke"`.
- **Theme developers / site owners:** No action required for the Search block. The front-end markup is unchanged, and `.wp-block-search__button svg` `fill` rules still color the button icon in both the editor and front end.
- **Version skew:** The icons CHANGELOG notes that new bundled stroke icons paired with an older externalized `wp.components.Icon` can lose merged styles when an unrelated `style` prop is passed. Update the paired packages together.

## Technical details

**`packages/icons/src/library/search.svg`** changes from a filled path (`fill="currentColor"`) to a stroked path:

```svg
<!-- before -->
<svg viewBox="0 0 24 24" fill="currentColor"><path d="M13 5c-3.3 0-6 2.7-6 6 ...z" /></svg>

<!-- after -->
<svg viewBox="0 0 24 24" style="fill: none" stroke="currentColor" stroke-width="1.5">
  <path d="M5 19L9.28769 14.7123M18.25 11C..." vector-effect="non-scaling-stroke" />
</svg>
```

This follows the stroke conventions from #78808, which declares `fill: none` via inline `style` so ordinary third-party CSS like `.foo svg { fill: currentColor }` doesn't override it.

**`packages/block-library/src/search/edit.jsx`** stops importing `search` from `@wordpress/icons` (still imports `Icon`). It defines a module-level `searchBlockIcon`, an `SVG`/`Path` element from `@wordpress/primitives` that carries the original filled path data, and renders `<Icon icon={ searchBlockIcon } />` in the "Use button with icon" button. The PHP renderer is untouched, so editor and front end remain aligned. A code comment flags that this copy must stay in sync with the PHP renderer.

CHANGELOG entries were added to both `packages/icons` (Breaking Changes) and `packages/block-library` (Internal). No hooks, REST, or `block.json` changes.

## Contribution

Follow-up to #78812 (which prepared the stroke version) and #78808 (stroke conventions), authored by @ciampo with OpenAI Codex assistance disclosed. @jasmussen endorsed decoupling the block icon from `@wordpress/icons`: the two will drift, but `@wordpress/icons` are WordPress UI icons, and the frontend blocks only reused them for speed. He suggested a separate future effort to refactor those blocks onto the Icon block so themes can register their own icon sets or swap which icon is used where.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
