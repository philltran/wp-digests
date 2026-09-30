# #82573: Text: Migrate Emotion styles to an SCSS Module

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Components`
- **Merged:** [`bae0ff5`](https://github.com/WordPress/gutenberg/commit/bae0ff5a47907c94a2b22cf477d2b4748490a47b)
- **Discussion:** [#82573](https://github.com/WordPress/gutenberg/pull/82573) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `Text` component in `@wordpress/components` is migrated from Emotion CSS-in-JS to a static SCSS module (`style.module.scss`). Dynamic typography values (color, font-size, font-weight, line-height, letter-spacing, text-align, display) are now applied as CSS custom properties (`--wp-components-text-*`) on the element's inline `style`, while fixed styles (destructive color, highlighter `mark` styling, block display, muted color, uppercase) move to named classes. This is part of the broader effort to remove Emotion from `@wordpress/components` (tracked in #66806). The migration also fixes two cascade-order bugs: single-line `truncate` now consistently applies `white-space: nowrap`, and `isBlock` consistently disables multi-line clamping, in both the main document and iframes.

## Impact

- **Plugin & theme developers using `<Text>` or `<Heading>`:** No action required for standard usage. Public props, rendered element, and visual output are preserved. However, if you compose source-order-dependent Emotion style fragments with `cx()` and pass them as separate arguments, the override order may change. Pass such fragments in a single `css()` call instead.
- **Consumers of `PaletteEdit` and `DatePicker`:** Their `Heading` overrides (`NavigatorHeading`, `PaletteHeading`) were updated internally to use `&&` / `&&&` specificity to win against the new module's typography. No consumer-facing change.
- **Anyone importing from `@wordpress/components` internals:** `packages/components/src/text/styles.ts` (the old Emotion `css` exports: `Text`, `block`, `positive`, `destructive`, `muted`, `highlighterText`, `upperCase`) is deleted. The `useCx` hook is no longer used by `Text`. The `COLORS` import from `../utils` is removed from the text hook.
- **Site owners / hosting:** No action required. Bundle size increases by ~1.8 kB in `build/scripts/components/index.min.js`.

## Technical details

The core rewrite is in `packages/components/src/text/hook.ts`. The old approach built an Emotion `css()` object from `styles.Text`, a `base` object, optional `optimalTextColor`, and conditional variant styles, then merged with `cx()`. The new approach:

1. Builds a `typography` plain object with kebab-case keys (`font-size`, `font-weight`, `line-height`, `letter-spacing`, `text-align`, `color`, `display`).
2. Iterates entries: if a value is a CSS-wide keyword (`inherit`, `initial`, `unset`, `revert`, `revert-layer`), it pushes a dedicated class (e.g. `styles['font-size-inherit']`) because CSS-wide keywords cannot be assigned to a custom property. Otherwise it sets `--wp-components-text-<property>` on a `textStyle` object.
3. Composes the final `className` with `clsx` from `styles.text`, conditional modifier classes (`.has-display`, `.has-letter-spacing`, `.has-text-align`, `.readability-dark`, `.readability-light`, `.destructive`, `.highlighter-text`, `.block`, `.muted`, `.upper-case`), keyword classes, and the consumer's `className`.
4. Merges the consumer's `style` prop: `style: { ...textStyle, ...style }`.

The new `style.module.scss` resets all seven custom properties to `initial` on `.text` so nested `<Text>` elements don't inherit a parent's typography. `text-wrap: pretty` is placed in `:where(.text)` to keep specificity at 0. Optional declarations (`display`, `letter-spacing`, `text-align`) are gated behind `.has-*` classes so they only emit when the prop is supplied.

In `date-picker/styles.ts`, `NavigatorHeading` wraps `font-size` and `font-weight` in `&&` to override the module's `var()`-based typography regardless of stylesheet order. In `palette-edit/styles.ts`, `line-height` moves into the existing `&&&` block alongside `font-size` for the same reason.

The old `styles.ts` (Emotion `css` template literals) is deleted. ESLint `no-restricted-imports` suppressions for both text files are removed from `tools/eslint/suppressions.json`.

Browser tests gain a `beforeEach`/`afterEach` pair that sets and removes `--wp-components-color-gray-700` on `document.documentElement`, because the SCSS now references theme tokens that the production build resolves but the test environment does not.

## Contribution

Opened by @ciampo as part of the Emotion-removal tracking issue #66806, with @mirka as co-author. The PR notes it was implemented and verified with Codex, including independent review by same-model subagents. The discussion is minimal (2 comments, 0 reactions), with no visible design debate or rejected alternatives in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
