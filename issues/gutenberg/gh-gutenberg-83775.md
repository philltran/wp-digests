# #83775: Update UI Spinner appearance and add a color prop

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`463e2a7`](https://github.com/WordPress/gutenberg/commit/463e2a7d6440ca1ca69fe614cae575e921475fe3)
- **Discussion:** [#83775](https://github.com/WordPress/gutenberg/pull/83775) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/ui` `Spinner` has a new look and a new `color` prop. It no longer draws a gray track under a blue quarter-circle. It now draws a single 180° arc, like the `Button` loading spinner. The arc defaults to the weak neutral content foreground. `color` takes any CSS color string, including `currentColor` to inherit the surrounding text color.

## Impact

- **Plugin/theme developers using `@wordpress/ui` `Spinner`:**
  - The default spinner looks different: no track, a longer arc, and a neutral color instead of brand blue. Check any visual snapshots or layouts that depend on the old appearance.
  - Pass `color` (for example `color="currentColor"`) to match the surrounding context or a colored background.
  - `color` is now typed as `string` and omitted from the inherited SVG props, so it is no longer the SVG `color` attribute type from `ComponentProps<'svg'>`.
- **Accessibility:** in forced-colors mode the arc uses `CanvasText`, so it stays visible against any background.
- **Site owners / editor users:** no direct action. Because the spinner now has no track and a neutral default color, it reads differently wherever UI components are used.
- No deprecations or removed exports.

## Technical details

Changes are confined to `packages/ui/src/spinner/`.

- **`spinner.tsx`**
  - The `<circle className={styles.track}>` element is removed.
  - The path changes from `d="m 50 0 a 50 50 0 0 1 50 50"` (quarter arc) to `d="m 50 0 a 50 50 0 0 1 0 100"` (half arc).
  - `color` and `style` are destructured from props. The `color` is merged into the inline style only when defined: `style={ color === undefined ? style : { ...style, color } }`.
  - The forwardRef generic changes from `ComponentProps<'svg'>` to the new `SpinnerProps`.
- **`types.ts` (new)**

```ts
export type SpinnerProps = Omit< ComponentProps< 'svg' >, 'color' > & {
	color?: string; // default: var(--wpds-color-foreground-content-neutral-weak)
};
```

- **`style.module.css`**
  - The `.track` rule is removed.
  - The root sets `color: var(--wpds-color-foreground-content-neutral-weak)`.
  - `.indicator` now uses `stroke: currentColor` and `stroke-width: var(--wpds-border-width-sm)` (previously a fixed `1.5px` and `--wpds-color-background-thumb-brand`).
  - A `@media (forced-colors: active)` rule sets `stroke: CanvasText`.
  - The spin animation is unchanged, and it is still intentionally kept under `prefers-reduced-motion`.
- **Stories and changelog**
  - Adds `CustomColor` and `CurrentColor` stories and a `color: { control: 'text' }` argType.
  - Adds a `packages/ui/CHANGELOG.md` entry under Enhancements.

The `Button` loading indicator is not changed here.

## Contribution

Opened and merged by @mirka, with @ciampo and @jameskoster credited. @ciampo suggested reusing the new spinner in `Button` in the same PR to compare before and after. @mirka kept that as a separate follow-up, though local testing looked fine. A storybook manifest issue and failing trunk tests came up late. @manzoorwanijk pointed to #83856 as the fix.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
