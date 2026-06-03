# #78614: Button.Icon: Fix clipped icons

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @macayu17
- **Labels:** `[Type] Bug`, `First-time Contributor`, `[Package] UI`
- **Merged:** [`bdf8db2`](https://github.com/WordPress/gutenberg/commit/bdf8db2cbf95a5235610d78636f063f393ae13a0)
- **Discussion:** [#78614](https://github.com/WordPress/gutenberg/pull/78614) · 3 comments · 0 reactions

## Summary

`Button.Icon` in `@wordpress/ui` was hardcoding `viewBox="4 4 16 16"` on every icon it rendered. This inset viewBox works only for icons whose artwork is centred in a 24×24 canvas with 4px padding on all sides; icons whose paths extend outside that region — including `wordpress`, `keyboardClose`, and others in `@wordpress/icons` — were visually clipped. The fix removes the forced `viewBox` override so each icon renders with its own declared viewBox, resizes the icon element to the standard 24px, and compensates for the size increase with a negative CSS margin (`calc((16px - 24px) / 2)`) so the icon's layout footprint inside the button remains 16px.

## Impact

**Plugin & theme developers / component library consumers**
- If you consume `Button.Icon` from `@wordpress/ui` and rely on the previous `size={16}` / `viewBox="4 4 16 16"` rendering, the rendered SVG will now be `24px` in both `width` and `height` with a `−4px` margin on all sides. Visual output should be equivalent for standard icons, but any CSS that targets the SVG's intrinsic dimensions will be affected.
- Icons with non-standard viewBoxes that were previously clipped now render correctly — no action required to benefit from the fix.
- No changes to the public TypeScript API for `Button.Icon`; the `className` prop (already part of `ButtonIconProps`) is now forwarded through to the underlying `Icon` element.
- **No breaking changes** for sites using the icons via the classic `@wordpress/components` `Button` component — this change is isolated to the newer `@wordpress/ui` package.

## Technical details

**File:** `packages/ui/src/button/icon.tsx`

Before:
```tsx
<Icon
  ref={ ref }
  icon={ icon }
  viewBox="4 4 16 16"
  size={ 16 }
  { ...props }
/>
```

After:
```tsx
<Icon
  ref={ ref }
  icon={ icon }
  className={ clsx( styles.icon, className ) }
  size={ 24 }
  { ...props }
/>
```

The `viewBox` prop is no longer passed, so the icon's own `viewBox` attribute is preserved. `size` is raised from `16` to `24` (matching the standard icon canvas). The `className` prop is now extracted from `props` and merged with a new CSS Module class `styles.icon` via `clsx`.

**File:** `packages/ui/src/button/style.module.css`

A new CSS custom property is declared inside the button's root rule block:
```css
--wp-ui-button-icon-margin: calc((16px - 24px) / 2); /* = -4px */
```
A new `.icon` selector (scoped inside the button module) applies that margin:
```css
.icon {
  margin: var(--wp-ui-button-icon-margin);
}
```
Both values are hardcoded with a `TODO` comment noting they should be replaced with design-system size tokens once available.

**Storybook story** (`packages/ui/src/button/stories/index.story.tsx`): the `WithIcon` story's icon is swapped from `cog` to `wordpress` — the `wordpress` icon uses a non-standard viewBox, making it a better regression canary.

A unit regression test was added at `packages/ui/src/button/test/icon.test.tsx` (not in the diff but referenced in testing instructions) to assert that a non-standard viewBox is not overridden.

## Contribution

Opened and implemented by first-time contributor @macayu17, who also filed the originating issue (#78595). @ciampo was requested as reviewer. The PR was merged as commit `bdf8db2`. Discussion in the thread was minimal (3 comments, bot-generated); no alternative approaches were recorded. The `TODO` comment in the CSS acknowledges a known limitation — the `−4px` margin is hardcoded because the design-system size token for this dimension does not yet exist.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
