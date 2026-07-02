# #75918: Navigation: Allow creating new links in site editor sidebar List View

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jeryj
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Package] Edit Site`, `[Feature] Site Editor`, `[Feature] Navigation in Site View`, `[Feature] Navigation Menus`
- **Merged:** [`93bfaf1`](https://github.com/WordPress/gutenberg/commit/93bfaf18c78acf68d5e83ed96173c3b60189bc60)
- **Discussion:** [#75918](https://github.com/WordPress/gutenberg/pull/75918) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Enables creating new navigation links directly from the Site Editor's sidebar List View, eliminating the need to switch to the canvas or block inspector for menu item management. This enhancement closes #66367 and expands menu capabilities within the global site editor flow.

## Impact

- **Site editors & designers**: Gains the ability to add pages, custom URLs, or newly created pages directly from the sidebar list view appenders.
- **Plugin/theme developers**: No breaking changes; relies on existing navigation block APIs and client-side state.
- **Action required**: None. Existing menu workflows, context menus, and canvas transitions remain fully functional and backward-compatible.

## Technical details

- Exports `NavigationLinkUI` as a private API and mounts it within the site editor's sidebar List View to handle link creation interactions.
- Implements search-as-you-type page selection, external URL entry, and dynamic page creation workflows triggered from the list view appender.
- Adds end-to-end test coverage in `test/e2e/specs/site-editor/navigation-sidebar-list-view.spec.js` validating escape dismissal (no empty items), focus trapping, and page submission flows.
- Client-side only; no database schema, REST routes, or block.json fields are modified. Bundle size impact is negligible (+84 B).

## Contribution

Opened by @jeryj as part of a broader site editor navigation enhancement initiative (#75920) to address limited sidebar capabilities (Closes #66367). Merged at `93bfaf1` after passing automated checks. Flaky test reports generated during review were unrelated (`router-regions.spec.ts`) and did not impact this change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
