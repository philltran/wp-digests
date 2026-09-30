# #82242: Global styles: resolve theme-relative background image URLs for display

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `Global Styles`
- **Merged:** [`87dfece`](https://github.com/WordPress/gutenberg/commit/87dfece01da3e02e5e63dd0dbfd1f4b16e2e0cb7)
- **Discussion:** [#82242](https://github.com/WordPress/gutenberg/pull/82242) · 12 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Theme-relative background image URLs (`file:./…`) and `ref` pointers in theme.json now resolve correctly in the Site Editor's Global Styles background controls. Since the Global Styles UI package refactor (#72599), the thumbnail, filename label, and focal point picker received the raw unresolved path (e.g. `file:./img/image.jpg`) and rendered broken. A new `useStyleWithResolvedBackground` hook resolves the `background` sub-tree before it reaches the panels.

## Impact

- **Theme developers:** Themes that set `styles.background.backgroundImage.url` (root or per-block, e.g. `core/group`) to a `file:./…` path will again show the image thumbnail, filename, and focal point picker in Styles → Background and Styles → Blocks → *block*. No theme changes required.
- **Plugin developers:** No API changes. The post editor's block inspector is unchanged.
- **Site owners:** Visual/UI fix in the Site Editor only; stored global styles keep their theme-relative paths.
- **Action required:** None.

## Technical details

**Root cause:** The background panels resolve `file:` URLs and `ref` pointers against Symbol-keyed block editor settings. Since #72599 the Global Styles screens render inside the `global-styles-ui` package's own public `BlockEditorProvider`, whose settings handling drops Symbol keys, so those settings were unavailable. The package is bundled and cannot use the private-API provider, and the PR treats the stripping as intentional, so it resolves at the data layer instead.

**Diff:**
- `packages/global-styles-ui/src/hooks.ts`: adds `useStyleWithResolvedBackground( style )`. It reads `merged` from `GlobalStylesContext` (which carries `_links` for theme files) and builds a tree of `{ styles, _links }`. It then iterates over `style.background` entries and calls `getResolvedValue` from `@wordpress/global-styles-engine` on each. Object values are shallow-copied first because `getResolvedValue` writes the resolved URL onto the object it receives, so the stored config keeps its theme-relative path. The result is memoized on `[ style, merged ]`; it returns `style` unchanged when there is no `background`.
- `background-panel.tsx` (root Styles → Background) and `screen-block.tsx` (per-block): the `inheritedValue` passed to `StylesBackgroundPanel` is now the resolved style instead of the raw `inheritedStyle`.
- `CHANGELOG.md`: adds a Bug Fixes entry.

In discussion, `getResolvedValue` was confirmed to call `getResolvedRefValue` internally, so `ref` pointers are covered without exporting anything additional. Bundle size impact was small (+12 B in edit-site, +125 B in editor).

## Contribution

Authored by @ramonjd with @andrewserong reviewing; the only design discussion was whether `ref` pointers would need `getResolvedRefValue` exported, which turned out to be unnecessary. After merge, @manzoorwanijk reported failing trunk unit tests. @ramonjd attributed this to an unrelated time-dependent dataviews DateTime test hardcoded to `2026-08-15` that broke when September began, and opened #82282 to fix it.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
