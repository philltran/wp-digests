# #80646: Input: Hide native spin controls for `type="number"`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`c66fc21`](https://github.com/WordPress/gutenberg/commit/c66fc213f08693e153775af65944e743965938a8)
- **Discussion:** [#80646](https://github.com/WordPress/gutenberg/pull/80646) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/ui` `Input` component now hides native browser spin controls for `type="number"` fields by default. This aligns the design system with a direction that favors explicit custom steppers (via `prefix`/`suffix` slots) over relying on inconsistent native browser UI. It simplifies the component's default behavior and removes the need for consumers to manually override browser-specific spin button styles.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:** `InputControl` with `type="number"` will no longer render native increment/decrement arrows. If your code relied on the default browser steppers, you must now implement custom controls using the `prefix` or `suffix` slots.
- **Gutenberg block authors:** Existing blocks using `InputControl` for numeric fields will see the native steppers disappear. Most blocks already hide them or don't use them, so visual changes will be minimal.
- **No action required** for consumers who already hide native steppers or use custom stepper implementations.

## Technical details

The change lives in `packages/ui/src/form/primitives/input/style.module.css`. It adds a scoped rule for `&[type="number"]` that applies `appearance: textfield` and targets `::-webkit-outer-spin-button` and `::-webkit-inner-spin-button` with `appearance: none` and `margin: 0`. This replaces the inline `<style>` block previously required in the `NumberWithSteppers` Storybook story (`packages/ui/src/form/input-control/stories/index.story.tsx`), which manually applied `-moz-appearance: textfield` and webkit spin-button resets.

**Before (inline workaround):**
```tsx
<style>
  { `.my-number-with-steppers input[type='number'] { -moz-appearance: textfield; }
     .my-number-with-steppers ::-webkit-inner-spin-button { appearance: none; }` }
</style>
```

**After (primitive stylesheet):**
```css
&[type="number"] {
  appearance: textfield;
  &::-webkit-outer-spin-button,
  &::-webkit-inner-spin-button {
    appearance: none;
    margin: 0;
  }
}
```
The CSS is now baked into the primitive, ensuring consistent behavior across browsers without consumer-side workarounds.

## Contribution

The PR addresses a design system direction discussed in issue #78335. The author noted that out of all `NumberControl` usages in Gutenberg, only three explicitly enabled custom spin buttons, while most either hid them or didn't need them. Rather than building a dedicated `NumberControl` with toggleable spin options, the team opted to hide native controls by default and push custom stepper composition to consumers via `prefix`/`suffix` slots. The diff reflects this simplified approach, stripping the inline CSS workaround from the storybook example and centralizing the reset in the primitive stylesheet.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
