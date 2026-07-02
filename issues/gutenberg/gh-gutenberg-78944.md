# #78944: UI: add LinkButton

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`7b5e419`](https://github.com/WordPress/gutenberg/commit/7b5e419ef4b7ee5e89b098e96b568e774e51248c)
- **Discussion:** [#78944](https://github.com/WordPress/gutenberg/pull/78944) · 14 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds the `LinkButton` component to the `@wordpress/ui` package to address UX ambiguity where developers need navigation (`<a>` semantics) with button styling. Includes updated usage guidelines clarifying when to choose `Button`, `IconButton`, `Link`, or `LinkButton` based on user intent.

## Impact

- Affects block/theme developers and plugin authors extending the Gutenberg design system via `@wordpress/ui`.
- No breaking changes; purely additive.
- Developers should default to standard `Link` for navigation unless button prominence is required, reserving `LinkButton` specifically for standalone calls-to-action or card footers.
- Requires importing `LinkButton` from `@wordpress/ui` rather than manually composing `Button` with an anchor.

## Technical details

- Introduces `packages/ui/src/link-button/` containing `index.ts`, `link-button.tsx`, `icon.tsx`, and `style.module.css`.
- Exported via `packages/ui/src/index.ts` as `LinkButton` and `LinkButton.Icon`.
- Implementation wraps the existing `<Link>` component with `variant="unstyled"` and applies CSS modules for button visual traits (`buttonStyles.button`, `is-{tone}`, `is-{variant}`, `is-{size}`).
- The `style.module.css` bridges button foreground tokens (`--wp-ui-button-foreground-color`) to the anchor element, explicitly handling `:visited`, `:hover`, `:active`, and `:focus` states.
- Supports standard link props including `openInNewTab`, but intentionally omits button-specific properties (`disabled`, `loading`, `nativeButton`, `focusableWhenDisabled`, `aria-pressed`).

## Contribution

Merged by @simison, with co-authors @jameskoster, @ciampo, @mirka, and @aduth. Discussion highlighted the blurred lines between `Button` and `Link` in React apps; the author added extensive usage guidelines to codify component selection based on user intent. Features like a `selected` state or icon support for `openInNewTab` were discussed but deferred to future iterations to ship the baseline pattern sooner.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
