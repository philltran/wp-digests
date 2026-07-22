# #78823: UI: Simplify focus ring styles

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`b983fd2`](https://github.com/WordPress/gutenberg/commit/b983fd28f4c2d9eee8bb63c1ea2a17e6ed59d962)
- **Discussion:** [#78823](https://github.com/WordPress/gutenberg/pull/78823) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Removes the shared `@wordpress/ui` focus ring transition and base outline styles, applying `outline` and `outline-offset` only in active focus states. This eliminates inconsistent focus ring animations caused by component-level transitions overriding the utility's shorthand.

## Impact

- **Plugin & theme developers** using `@wordpress/ui` components (e.g., `Button`, `Link`, form controls) will see focus rings appear instantly without a fade-in transition.
- **UI customizers** relying on the previous `transition: outline 0.1s ease-out` behavior for custom focus styles will need to adjust their CSS if they want to replicate the old animation.
- **No action required** for standard plugin/theme developers; this is a purely CSS/utility layer adjustment with no JS API changes.

## Technical details

The diff modifies `packages/ui/src/utils/css/focus.module.css` to strip the `@layer utilities` block that previously set `transition: outline 0.1s ease-out;`, `outline: 0 solid transparent;`, and `outline-offset: 1px;` on base focus selectors (`.outset-ring--focus`, `.outset-ring--focus-visible`, etc.). Instead, `outline` and `outline-offset` are now declared directly inside the active state selectors (e.g., `.outset-ring--focus:focus`). The `overlay-chrome.module.css` comment is updated to reflect that `outline-offset` is no longer animated by the removed transition.

## Contribution

The PR moved quickly through review with minimal discussion, focusing on resolving the transition override inconsistency. The record contains no notable design debates or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
