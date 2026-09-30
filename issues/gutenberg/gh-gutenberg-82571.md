# #82571: Grid: Migrate Emotion styles to SCSS Modules

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Components`
- **Merged:** [`577004d`](https://github.com/WordPress/gutenberg/commit/577004dfab6f8de7e3aeedb2fd0c61c96494b3e1)
- **Discussion:** [#82571](https://github.com/WordPress/gutenberg/pull/82571) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`Grid` in `@wordpress/components` no longer renders its styles through Emotion. Layout is now driven by a new `style.module.scss` with module classes, and dynamic values (gap, tracks, alignment) are passed as `--wp-components-grid-*` custom properties on the element's inline `style`. The change continues the effort to remove Emotion from the package (#66806). Props, responsive columns/rows, polymorphism and consumer overrides are meant to behave as before.

## Impact

**Plugin & theme developers**
- No API change to `Grid` props; no action required for typical usage.
- Emotion-based consumers that compose `Grid`'s styles with `cx()` are affected. The CHANGELOG lists this under *Breaking Changes*: source-order-dependent Emotion fragments should be passed in a single `css()` call, since separate fragments can change override order now that `Grid` no longer emits Emotion styles.
- Consumer stylesheets now keep precedence for `align-items`, `justify-content`, `grid-template-columns` and `grid-template-rows` when the matching prop is omitted, because those declarations are only applied when the prop is set. Passing a `style` prop overrides layout props.
- Anything targeting Emotion-generated `Grid` class names will no longer match; module class names are hashed.

**Users of dependent components**
- `BoxControl` and `ToolsPanel` are named in the testing instructions as `Grid` consumers and were checked for layout regressions (LTR/RTL).

**Bundle size**
- The bot reports `build/scripts/components/index.min.js` growing by about 1.76 kB (+0.66%).

## Technical details

**`grid/hook.ts`**
- Drops `@emotion/react`, `useMemo`, `useCx` and `CONFIG.gridBase`. The hook now builds `className` with `clsx`, using `styles.grid`, `is-inline`, and presence classes `has-align`, `has-justify`, `has-template-columns`, `has-template-rows`.
- Dynamic values go into inline custom properties: `--wp-components-grid-align`, `-justify`, `-gap`, `-template-columns`, `-template-rows`, `-row-gap`, `-column-gap`. The caller's `style` is now destructured and spread last, so it overrides them.
- Numeric `rowGap`/`columnGap` are converted to `px` through `toCSSValue()`.
- CSS-wide keywords (`inherit`, `initial`, `unset`, `revert`, `revert-layer`) would otherwise resolve on the custom property itself. The hook detects them and adds classes such as `row-gap-inherit`, which set the real property (e.g. `row-gap: inherit`).
- Fix: `gridTemplateColumns`/`gridTemplateRows` fallbacks now test the resolved `column`/`row` value rather than `columns`/`rows`.

**`grid/style.module.scss`**
- `.grid` resets all seven custom properties to `initial`, so nested Grids use only their own props, and sets `display: grid`. `.is-inline` gives `inline-grid` with `vertical-align: middle`.
- `row-gap` and `column-gap` are separate rules with fallbacks: `var(--wp-components-grid-row-gap, calc($grid-unit-05 * var(--wp-components-grid-gap)))`. The two axes are kept separate so production CSS sorting cannot reset an axis override.
- Presence-gated rules use `:not(:where(...keyword classes))`, and an `@each` loop generates the keyword classes.

**`grid/utils.ts`**
`getAlignmentProps` now uses `ALIGNMENTS[ alignment ] ?? {}`, so an unsupported runtime `alignment` value is ignored rather than yielding `undefined`.

**Other**
- Adds a `style composition` block to `grid.browser.test.tsx` (Vitest Browser Mode) covering fallback overrides, nested Grids, inline style precedence, consumer stylesheet precedence, CSS-wide values, and unsupported alignment.
- Removes the `grid/hook.ts` `no-restricted-imports` entry from `tools/eslint/suppressions.json`.

## Contribution

The PR follows the migration guide in #82567 and is part of the broader Emotion removal tracked in #66806. The description states that Codex authored the implementation, tests and description, with @ciampo as author and @mirka credited by props-bot. The record shows only bot comments and no design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
