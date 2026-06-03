# #78529: Navigation: Hard deprecate component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Type] Breaking Change`, `[Package] Components`, `[Package] Editor`, `Needs Dev Note`
- **Merged:** [`be000ae`](https://github.com/WordPress/gutenberg/commit/be000aed6631630bad8f29282cc2d124c6a1aa23)
- **Discussion:** [#78529](https://github.com/WordPress/gutenberg/pull/78529) · 2 comments · 0 reactions

## Summary

The legacy `Navigation` component and its associated subcomponents have been permanently removed from `@wordpress/components`. Originally deprecated in WordPress 6.8, this hard deprecation finalizes the removal of the deprecated API, as the `Navigator` component has fully replaced it for managing navigation states in editor toolbars and site structures.

## Impact

- **Plugin & Theme Developers**: Any remaining usage of `Navigation` or `__experimentalNavigation*` exports from `@wordpress/components` will break at runtime. You must migrate to `Navigator` and its modern primitives immediately.
- **Editor Consumers**: Stale CSS overrides targeting `.components-navigation*` class names have been purged from editor build assets, slightly reducing payload size without affecting current editor layout since the target component no longer renders.
- **Hosting & Platform Teams**: No migration required; this is a controlled API removal with an established deprecation timeline.

## Technical details

- Exports for `Navigation` and all `__experimentalNavigation*` symbols are stripped from `packages/components/src/index.ts`.
- The component's documentation, stories, tests, and manifest entry (`docs/manifest.json`) are completely removed from the package.
- Editor stylesheet assets (e.g., `build/styles/editor/style.css`) lose stale rules targeting `.components-navigation*`, contributing to a net ~3.6 kB bundle reduction.
- Migration pattern shift:
  ```javascript
  // Before (Removed)
  import { Navigation, __experimentalNavigationLink } from '@wordpress/components';
  
  // After (Required)
  import { Navigator } from '@wordpress/components';
  ```
- The removal is absolute; no compatibility stubs or no-op wrappers are provided to prevent masking broken plugin implementations.

## Contribution

PR #78529 was authored by @mirka and co-authored by @ciampo. It executed the scheduled hard deprecation timeline outlined since WP 6.8, choosing a complete codebase purge over a compatibility stub after determining negligible usage in the plugin directory. The change closed out the legacy navigation API cleanup without introducing new design alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
