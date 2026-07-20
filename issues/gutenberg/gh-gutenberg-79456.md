# #79456: Experiments: Move screen under Settings, drop top-level Gutenberg menu and Demo page

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @youknowriad
- **Labels:** `[Type] Enhancement`, `Gutenberg Plugin`, `No Core Sync Required`
- **Merged:** [`0ece78d`](https://github.com/WordPress/gutenberg/commit/0ece78d9b36340f19aa7a043e375951e5a4812eb)
- **Discussion:** [#79456](https://github.com/WordPress/gutenberg/pull/79456) · 16 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Gutenberg plugin removes its dedicated top-level wp-admin menu and relocates the Experiments screen to Settings > Gutenberg Experiments. The legacy Demo page and its associated fixture files are removed, but the demo content is preserved and now accessible via a new editor command palette entry. This consolidates plugin settings under the core Settings menu and aligns with modern WordPress admin navigation patterns.

## Impact

- **Site admins & plugin/theme developers:** The top-level **Gutenberg** admin menu is removed. Any custom scripts, bookmarks, or UI elements relying on `admin.php?page=gutenberg` or the old menu hierarchy will break.
- **Experiments users:** The Experiments screen is now under **Settings > Gutenberg Experiments** and requires the `manage_options` capability (administrator-only). Legacy URLs like `?page=gutenberg-experiments` now redirect to `options-general.php?page=experiments-wp-admin`.
- **Demo content users:** The old Demo page is gone. To insert the legacy demo post content, use the editor command palette (Cmd/Ctrl+K) and search for "Insert demo content".
- **No action required** for standard Gutenberg plugin usage beyond updating bookmarks or scripts that referenced the old admin path.

## Technical details

- `lib/init.php`: `gutenberg_menu()` no longer calls `add_menu_page()`. It now registers the Experiments screen via `add_submenu_page( 'options-general.php', ... )` with the page title "Gutenberg Experiments" and capability `manage_options`, pointing to the auto-generated `gutenberg_experiments_wp_admin_render_page` callback.
- `lib/experimental/experiments/load.php`: Removes the redundant `gutenberg_experiments_menu()` submenu registration. Updates `gutenberg_redirect_legacy_experiments_page()` to redirect legacy `?page=gutenberg-experiments` requests to `options-general.php?page=experiments-wp-admin`.
- `lib/demo.php`: Refactored to remove the old redirect and `default_content`/`default_title` filters. Introduces `gutenberg_demo_content()` to return serialized block markup, and `gutenberg_enqueue_demo_command()` which registers a `core/commands` command (`gutenberg/insert-demo-content`). The command injects blocks via `wp.data.dispatch( 'core/block-editor' ).insertBlocks()` using an inline `window.gutenbergDemoContent` variable.
- `bin/build-plugin-zip.sh` & `docs/`: Removes `post-content.php` from the build zip and updates documentation paths to reflect the new Settings menu location.

**Before/After (Admin Menu Registration):**
```php
// Before
add_menu_page( __( 'Gutenberg', 'gutenberg' ), __( 'Gutenberg', 'gutenberg' ), 'edit_posts', 'gutenberg', 'gutenberg_experiments_wp_admin_render_page' );
add_submenu_page( 'gutenberg', __( 'Experiments Settings', 'gutenberg' ), __( 'Experiments', 'gutenberg' ), 'manage_options', 'experiments-wp-admin', 'gutenberg_experiments_wp_admin_render_page' );

// After
add_submenu_page( 'options-general.php', __( 'Gutenberg Experiments', 'gutenberg' ), __( 'Gutenberg Experiments', 'gutenberg' ), 'manage_options', 'experiments-wp-admin', 'gutenberg_experiments_wp_admin_render_page' );
```

## Contribution

Opened and merged by @youknowriad with co-authors @tyxla, @fabiankaegy, @ellatrix, @Mamaduka, and @mrfoxtalbot. The change originated from reviewer feedback suggesting the top-level menu was unnecessary. The team agreed to consolidate under Settings, while preserving the demo content via the command palette rather than deleting it entirely. Documentation paths were updated concurrently to match the new navigation structure.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
