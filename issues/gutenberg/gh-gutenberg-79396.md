# #79396: Command Palette: Fix duplicate enqueue breaking the palette in the Site Editor

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Feature] Command Palette`
- **Merged:** [`50bf905`](https://github.com/WordPress/gutenberg/commit/50bf905edf3892afee7575ac20cf49c1a3b3ba28)
- **Discussion:** [#79396](https://github.com/WordPress/gutenberg/pull/79396) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Command Palette now opens correctly in the Extensible Site Editor and other boot-based admin pages after a regression caused by duplicate script enqueues. A previous cleanup removed a compatibility layer that previously prevented `wp_enqueue_command_palette_assets` from firing twice, which split the `core/commands` store and broke the UI. This patch restores single-enqueue behavior by explicitly removing the core action during asset capture.

## Impact

- **Plugin & theme developers:** No action required. The fix is internal to the block editor settings controller and does not expose new public APIs or require code changes.
- **Site owners & editors:** Restores expected `<kbd>Cmd</kbd>/<kbd>Ctrl</kbd>+<kbd>K</kbd>` functionality in the Site Editor without manual intervention.
- **Hosting & platform teams:** No configuration changes needed; the fix applies automatically with the next Gutenberg/Core release.
- **Headless & REST consumers:** No impact. The `/wp-block-editor/v1/assets` endpoint continues to return the same asset manifest, now without triggering duplicate script loads on the client.

## Technical details

The change modifies `lib/experimental/class-wp-rest-block-editor-settings-controller.php` inside the `get_assets()` method. It adds `remove_action( 'admin_enqueue_scripts', 'wp_enqueue_command_palette_assets' );` before the existing `remove_action` calls that strip unwanted scripts during the REST asset capture. Previously, the removal relied on a dropped compat file that swapped the core function; without it, `wp_enqueue_command_palette_assets` remained attached, causing `wp-core-commands` to load twice and instantiate two separate Redux stores. The diff ensures the action is detached during the `/wp-block-editor/v1/assets` endpoint execution, preventing duplicate script tags and store fragmentation.

## Contribution

Opened as a follow-up to #79196, the PR was merged after a brief review cycle. @youknowriad questioned why the auto-load logic failed to detect the already-loaded assets, prompting @t-hamano to note that the auto-load detection may need future refinement, but the explicit `remove_action` was accepted as the immediate fix. The change was co-authored by @youknowriad and @fushar.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
