# #78820: UI Button: Fix loading state in forced colors

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] UI`
- **Merged:** [`070c50f`](https://github.com/WordPress/gutenberg/commit/070c50faed75660eb6a1f81a236207e4c33f2c6f)
- **Discussion:** [#78820](https://github.com/WordPress/gutenberg/pull/78820) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/ui` Button component now correctly hides its loading text and renders its spinner in forced-colors environments. Previously, the component relied on `color: transparent` to hide the loading label, which forced-colors remaps to a visible system color, causing the text to appear behind the spinner. The fix applies a `forced-colors` media query override to switch the text color to `ButtonFace` and adjusts the spinner borders to use `ButtonText`, ensuring the loading state remains visually correct for users relying on high-contrast or forced-color settings.

## Impact

- Plugin & theme developers using `@wordpress/ui` Button with the `loading` prop will see corrected accessibility behavior in forced-colors environments.
- The fix explicitly excludes the `unstyled` variant, so existing `unstyled` button implementations remain unaffected.
- No breaking changes, deprecations, or migration steps required.

## Technical details

The change modifies `packages/ui/src/button/style.module.css`. It scopes the loading state to `.is-loading:not(.is-unstyled)` to prevent the `loading` prop from inadvertently affecting `unstyled` buttons. A `@media ( forced-colors: active )` block is added to override the default `color: transparent` with `color: ButtonFace`, preventing forced-color remapping. The spinner's forced-colors override sets `border-color: ButtonText` and removes `border-block-end-style` and `border-inline-start-style` to avoid relying on transparent borders, which forced-colors can render unpredictably.

```css
/* Before */
.is-loading {
  color: transparent;
}

/* After */
.is-loading:not(.is-unstyled) {
  color: transparent;
  @media ( forced-colors: active ) {
    color: ButtonFace;
  }
}
```
No new hooks, filters, or block APIs are introduced; this is a pure CSS adjustment to the component's internal stylesheet.

## Contribution

During review, alternative approaches were proposed to hide the loading text without relying on color overrides, including using the `Field` system color, applying `text-indent` with `overflow: clip`, or re-introducing a nested `<span>`. The author evaluated these and opted for the color-only override, noting that `text-indent`/`overflow` strategies could conflict with consumer-defined truncation or overflow settings, and that avoiding DOM changes aligned with previous refactoring efforts. The final implementation applies only CSS media queries, keeping the bundle size impact minimal.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
