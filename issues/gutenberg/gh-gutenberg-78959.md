# #78959: UI: Update CSS cascade layers to use nesting

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`a716f1d`](https://github.com/WordPress/gutenberg/commit/a716f1d413fc87c48009d0f1e44117cba3be1dd1)
- **Discussion:** [#78959](https://github.com/WordPress/gutenberg/pull/78959) · 4 comments · 0 reactions

## Summary

Updates the CSS cascade layer structure in `@wordpress/ui` from a flat list of four prefixed layers (`wp-ui-utilities`, `wp-ui-components`, `wp-ui-compositions`, `wp-ui-overrides`) to a single top-level nested hierarchy under `wp-ui`. This refactor improves extensibility for applications defining custom cascade order, reduces layer ordering errors by centralizing the public-facing layer name, and better scopes shared style hierarchy within the package.

## Impact

- **Plugin & Theme Developers**: Must update any external CSS overrides targeting `@wordpress/ui` layers. Replace direct references to prefixed sub-layers (e.g., `@layer wp-ui-components`) with the new nested structure using the single public layer: `@layer wp-ui, ...`.
- **Application Developers**: When defining custom cascade order relative to `@wordpress/ui`, apply `@layer wp-ui, example-app;` instead of ordering individual sub-layers.
- **Platform/Hosting**: No action required. Visual styling remains identical with zero expected regressions.

## Technical details

The diff updates component stylesheets across `packages/ui/src/` (e.g., `style.module.css` for `alert-dialog`, `badge`, `button`). Each file previously declared a flat layer list at the top:
```css
@layer wp-ui-utilities, wp-ui-components, wp-ui-compositions, wp-ui-overrides;
```
This is replaced by a nested declaration wrapping all subsequent component styles:
```css
@layer wp-ui {
    @layer utilities, components, compositions, overrides;
    /* ... remaining styles ... */
}
```
Documentation in `packages/ui/README.md` and `CONTRIBUTING.md` is updated to reflect that sub-layers are now namespaced under `wp-ui` (e.g., `utilities`, `compositions`). The docs also clarify how to resolve bare element selector conflicts by adjusting cascade order relative to the single `wp-ui` layer. Internal variables, keyframes, and shared utility styles remain encapsulated within the new structure without changing runtime behavior.

## Contribution

Opened and merged by @aduth, with co-authors @ciapo and @mirka noted in commit metadata. The change closes #78695, implementing a proposed nesting strategy to improve layer scoping and extensibility. The diff primarily consists of mechanical indentation updates across component stylesheets to implement the nested syntax, resulting in a negligible bundle size decrease (-29 B).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
