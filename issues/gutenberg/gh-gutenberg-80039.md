# #80039: Checkbox: Add form primitive to @wordpress/ui

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`05cf0bc`](https://github.com/WordPress/gutenberg/commit/05cf0bc190e879c35cba5a34722703319f75e17e)
- **Discussion:** [#80039](https://github.com/WordPress/gutenberg/pull/80039) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/ui` package now exports a `Checkbox` form primitive built on `@base-ui/react/checkbox`. This component provides a styled, accessible checkbox with support for checked, disabled, and indeterminate states, establishing the foundation for higher-level controls like `CheckboxControl` and `CheckboxGroup`. Developers can now consume this primitive directly in React-based WordPress projects.

## Impact

- **Plugin & theme developers:** No action required. The component ships with a `use-with-caution` status, indicating it is not yet fully recommended for production alongside `@wordpress/components` until style consistency is finalized.
- **Block & headless developers:** Can now import `Checkbox` from `@wordpress/ui` for form inputs without relying on legacy component libraries.
- **No breaking changes, deprecations, or migration steps.**

## Technical details

The diff introduces `packages/ui/src/form/primitives/checkbox/checkbox.tsx`, which wraps `@base-ui/react/checkbox` using `forwardRef` to expose an `HTMLSpanElement`. It accepts `className` and `indeterminate`, spreading remaining props to `_Checkbox.Root`. The indicator dynamically swaps between `check` and `reset` icons from `@wordpress/icons` based on `state.indeterminate`. Styling uses `style.module.css` with CSS layers (`@layer wp-ui`) and design tokens like `--wpds-color-background-interactive-brand-strong`. The component is exported from `packages/ui/src/form/primitives/index.ts`, making it available as:

```tsx
import { Checkbox } from '@wordpress/ui';
```

Props are typed via `CheckboxProps`, which maps to `ComponentProps< typeof _Checkbox.Root >`.

## Contribution

Opened and merged by @mirka, with co-authorship from @ciampo. The PR was reviewed and merged with minimal discussion, focusing on delivering the foundational primitive. Storybook metadata explicitly notes the component is pending a broader style consistency review with `@wordpress/components` (tracked in #76135), which is why it ships with a `use-with-caution` status rather than a full production recommendation.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
