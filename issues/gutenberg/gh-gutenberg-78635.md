# #78635: UI Button: Fix pressed disabled styles for neutral minimal

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Bug`, `[Package] UI`
- **Merged:** [`9731b78`](https://github.com/WordPress/gutenberg/commit/9731b78b7e478d6255ae507d6fd68ae26b33a1d2)
- **Discussion:** [#78635](https://github.com/WordPress/gutenberg/pull/78635) · 3 comments · 0 reactions

## Summary

A visual bug in the `@wordpress/ui` `Button` component caused neutral minimal buttons with `aria-pressed="true"` to render the wrong disabled appearance. Pressed neutral minimal buttons adopt the "strong" visual style (visually matching a solid neutral button), but when simultaneously disabled, the component was incorrectly applying the "weak minimal" disabled design tokens rather than the "strong" disabled tokens. The result was a pressed+disabled button that looked identical to an unpressed disabled one. This fix corrects token resolution so the pressed+disabled state draws from the strong disabled appearance, consistent with a disabled solid neutral button.

## Impact

**Plugin & theme developers using `@wordpress/ui`**
- If you render `<Button variant="neutral">` (or equivalent minimal neutral) with both `aria-pressed="true"` and `disabled`, the visual output now correctly reflects the pressed+disabled state with the strong disabled appearance.
- No API changes, no deprecations, no prop interface changes. This is a purely visual/style correction.
- No action required — the fix is passive once the updated `@wordpress/ui` package version ships.

## Technical details

The `@wordpress/ui` `Button` component maps pressed neutral minimal buttons to the "strong" visual token set (the same one used by solid neutral buttons). Prior to this fix, the `:disabled` styles scoped to that pressed-neutral-minimal selector were still resolving from the "weak minimal" disabled tokens — the same tokens an unpressed minimal button uses when disabled — rather than the strong disabled tokens.

The fix corrects the CSS so that when a neutral minimal button carries `aria-pressed="true"` and is disabled, its disabled styles are drawn from the strong token set. The corrected state now visually matches a disabled solid neutral button, which is the intended design-system behavior.

The provided diff only surfaces the `packages/ui/CHANGELOG.md` addition:
```
- `Button`: Fix disabled and hover styles for neutral minimal buttons with `aria-pressed="true"` (#78635)
```
The underlying CSS token correction is in the `@wordpress/ui` package's button stylesheet (not included in the diff excerpt). Bundle size impact is -8 B in `build/modules/content-types/index.min.js`.

## Contribution

Opened and merged by @mirka. @ciampo is credited as a reviewer via the props-bot co-author attribution. The three PR comments are entirely automated (size-check bot, props bot, flaky-test reporter); no substantive design debate or alternative approaches are recorded in the provided discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
