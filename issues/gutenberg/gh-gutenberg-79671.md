# #79671: UI: Add Skeleton component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Enhancement`, `[Feature] UI Components`, `[Package] UI`
- **Merged:** [`c3c95dd`](https://github.com/WordPress/gutenberg/commit/c3c95dd3cd4095a896551fb7b2a8e19a51e4b4e6)
- **Discussion:** [#79671](https://github.com/WordPress/gutenberg/pull/79671) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Adds a new `Skeleton` component to the `@wordpress/ui` package to provide a consistent, accessible loading placeholder across the block editor. The component renders a pulsing div that adapts to WP Design System tokens and respects system `prefers-reduced-motion` preferences, allowing plugin and theme authors to standardize UI state transitions without building ad-hoc loaders.

## Impact

- **Block Editor plugin & theme developers:** Gain a standardized, accessible component for representing loading states, reducing reliance on inline spinner markup or inconsistent CSS hacks.
- **Internal editor maintainers:** Can replace existing loader patterns with the unified `@wordpress/ui/Skeleton` to align with the WP Design System token scale and improve perceived performance on slower connections.
- **Site owners & non-editor workflows:** No impact; this is a package-only addition requiring no migration or configuration changes.

## Technical details

The diff introduces `packages/ui/src/skeleton/` containing `index.ts`, `types.ts`, `skeleton.tsx`, and `style.module.css`. The component uses `forwardRef` from `@wordpress/element` and `useRender` from `@base-ui/react` to render a default `<div>`. Styles are scoped inside `@layer wp-ui`, applying a background color via `var(--wpds-color-background-surface-neutral-weak)` and an animation keyframe `skeleton-pulse` (a 1.5s opacity shift between 1.0 and 0.4). The pulse media query is explicitly guarded by `@media not ( prefers-reduced-motion )`. Accessibility defaults to `aria-hidden="true"` to mark it as decorative, while a `forced-colors: active` media query forces a visible border via the `CanvasText` token to prevent the placeholder from blending into the page background. Sizing props extend `ComponentProps< 'div' >` in `types.ts`. The component is exported in `packages/ui/src/index.ts` and added to the `use-recommended-components.js` ESLint allowlist.

```tsx
// Standardized loading state pattern
import { Skeleton } from '@wordpress/ui';

const LoadingRegion = () => (
  <div aria-busy="true" className="editor-loading">
    <Skeleton style={{ width: 'var(--wpds-dimension-surface-width-xs)' }} />
  </div>
);
```

## Contribution

Opened by `@juanfra` to resolve issue #74122, the PR was merged after review from `@simison`, `@ciampo`, `@mirka`, and `@fcoveram`. Reviewers acknowledged the design trade-offs of lazy-loading but agreed a unified skeleton improves UX consistency across varying infrastructures. A secondary discussion proposed expanding official usage guidelines for other loaders (spinner, progress bar) and optimistic UI patterns, which was explicitly scoped out of this PR to be addressed in follow-up documentation.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
