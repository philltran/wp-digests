# #78899: Element: add polyfills for render, hydrate, unmountComponentAtNode

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jsnajdr
- **Labels:** `[Type] Bug`, `Backwards Compatibility`, `[Package] Element`
- **Merged:** [`f2eb697`](https://github.com/WordPress/gutenberg/commit/f2eb69745ef93956ef225439e45e410594961959)
- **Discussion:** [#78899](https://github.com/WordPress/gutenberg/pull/78899) · 11 comments · 0 reactions

## Summary

Addresses #78827 by implementing polyfills for `render`, `hydrate`, and `unmountComponentAtNode` to restore backwards compatibility for legacy React DOM mounting APIs. This prevents runtime failures in environments where these methods are missing or have been altered by upstream React changes.

## Impact

- **Plugin & Theme Developers**: No breaking changes. The polyfills ensure that legacy React DOM mounting methods function reliably. If your code relies on `render`, `hydrate`, or `unmountComponentAtNode`, they will now execute without throwing reference errors.
- **Third-Party Integrations**: External plugins that import directly from `react-dom` (e.g., Elementor) are supported, as the polyfills are injected into the bundled vendor script.
- **Action Required**: None. The changes are backwards compatible and automatically applied to existing bundles.

## Technical details

- Adds polyfill implementations for `render`, `hydrate`, and `unmountComponentAtNode` inside the `@wordpress/element` package.
- Exposes these functions via a new dedicated entry point: `/react-polyfill`.
- Mirrors the implementation into `build/scripts/vendors/react-dom.min.js` to ensure compatibility for external consumers importing directly from `react-dom`. The `react-dom` variant intentionally omits `wp-deprecated` warning calls to avoid a build-time dependency on `wp-deprecated`, which the author noted as an outstanding issue requiring a future structural fix.
- Bundle impact: `+952 B` total (`build/scripts/element/index.min.js` +253 B, `build/scripts/vendors/react-dom.min.js` +699 B).

## Contribution

Opened and merged by @jsnajdr to resolve #78827. The pull request passed review with minimal friction, focusing on the negligible size overhead (+0.01% of total bundle). A key design trade-off involved excluding deprecation warnings from the `react-dom` polyfill to prevent circular dependencies with `wp-deprecated`; this was documented as an acknowledged limitation rather than a resolved architectural change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
