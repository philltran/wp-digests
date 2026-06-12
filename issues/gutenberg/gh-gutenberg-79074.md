# #79074: Rename `Toolbar in editor` experiment to match iteration issue

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Enhancement`, `[Package] Edit Post`, `[Package] Edit Site`
- **Merged:** [`5d26ee7`](https://github.com/WordPress/gutenberg/commit/5d26ee7755aefb46e08ccb705a52ecc910c686b5)
- **Discussion:** [#79074](https://github.com/WordPress/gutenberg/pull/79074) · 4 comments · 0 reactions

## Summary

Renamed the "Toolbar in editor" Gutenberg experiment to "Omnipresent Toolbar" (internal ID `gutenberg-omnibar`). This change aligns the experiment's terminology with its broader iteration issue, clarifying that it encompasses more than just showing the admin bar in fullscreen editors. The underlying functionality and UI behavior remain unchanged; only the identifiers and internal references have been updated.

## Impact

- **Plugin & theme developers**: Update any calls to `gutenberg_is_experiment_enabled()` or REST API queries that reference the previous `gutenberg-admin-bar-in-editor` slug to the new `gutenberg-omnibar` ID.

- **Site owners**: No action required. The experiment toggle in Gutenberg -> Experiments will display the updated label, but the behavior and UI remain identical.

- **Core team / contributors**: E2E test suites relying on the old experiment ID must be updated to avoid test failures.

## Technical details

Renamed `lib/experimental/admin-bar-in-editor/load.php` to `lib/experimental/omnibar/load.php` and updated the corresponding require path in `lib/load.php`. Callback functions were renamed throughout: `gutenberg_enable_admin_bar_in_editor_experiment()` became `gutenberg_enable_omnibar_experiment()`, `gutenberg_admin_bar_in_editor_body_class()` became `gutenberg_omnibar_body_class()`, and `gutenberg_enable_admin_bar_in_site_editor_v2()` became `gutenberg_enable_omnibar_in_site_editor_v2()`. Experiment registration in `lib/experimental/experiments/load.php` now uses `id => 'gutenberg-omnibar'`. SCSS imports were updated in `packages/boot/src`, `packages/edit-post/src`, and `packages/edit-site/src`. E2E fixtures in `test/e2e/specs/editor/various/fullscreen-mode.spec.js` now pass `'gutenberg-omnibar'` to `requestUtils.setGutenbergExperiments()`.

## Contribution

Opened by @fushar and merged at commit `5d26ee7`, this pull request was a straightforward refactor to align the experiment's codename with its tracking issue. Co-authored by @arthur791004, the PR systematically renamed files, functions, SCSS imports, and test fixtures without altering behavior or introducing new logic.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
