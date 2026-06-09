# #78814: Storybook: Enhance Theme Provider example with admin-ui Page.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `Storybook`, `[Package] Private APIs`, `Design System`
- **Merged:** [`1946796`](https://github.com/WordPress/gutenberg/commit/1946796a059b64a08922a1c99a9ca31c43b8ffa1)
- **Discussion:** [#78814](https://github.com/WordPress/gutenberg/pull/78814) · 3 comments · 0 reactions

## Summary

Updates the Storybook `ThemeProvider` example application to render a realistic WordPress admin layout instead of hand-rolled markup. Replaces custom headers with the `Page` component from `@wordpress/admin-ui` and adds a `withRouter` decorator to enable functional breadcrumb navigation. This provides an accurate, production-ready visual reference for how `ThemeProvider` seed colors propagate across both `@wordpress/ui` and `@wordpress/admin-ui` components.

## Impact

- **Storybook & Design System Maintainers:** Improves example fidelity by aligning mock layouts with actual component APIs. No breaking changes to existing Storybook configuration or exports.
- **Plugin/Theme Developers & Site Owners:** No action required. This is an internal documentation and visual regression test update within the Gutenberg monorepo.

## Technical details

Modifies the `ThemeProvider` example story to swap custom layout markup for `Page` from `@wordpress/admin-ui`, leveraging its full header API (visual icon, breadcrumbs, subtitle, badges, actions). Introduces a `withRouter` Storybook decorator to resolve navigation context for breadcrumb links. Applies design tokens to wrap the application mock in a bordered frame with large corner radius. The canvas controls for Primary and Background color pickers now correctly cascade theme tokens across both UI and admin-ui component groups in real time.

## Contribution

Opened and merged by @jameskoster (commit `1946796`). Review discussion was minimal (3 comments, 0 reactions), with no alternative approaches proposed or rejected. The change was accepted as a direct fidelity improvement to the Design System's example application without requiring architectural changes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
