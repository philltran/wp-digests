# #76559: Block visibility: add theme json opt out

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Design Tools`
- **Merged:** [`1a343f8`](https://github.com/WordPress/gutenberg/commit/1a343f8cdfb348862e17e6024b70726d6f381dbd)
- **Discussion:** [#76559](https://github.com/WordPress/gutenberg/pull/76559) · 9 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

Adds `settings.blockVisibility.allowEditing` to `theme.json`, allowing themes to disable the block visibility UI entirely. When set to `false`, the toolbar toggle, block options menu item, command center entry, and keyboard shortcut are hidden, while preserving any previously saved visibility attributes on the blocks. This brings `blockVisibility` in line with other block supports like `color` and `typography` that can be opted out of at the theme level.

## Impact

- **Theme developers:** Can hide visibility controls by adding `"settings": { "blockVisibility": { "allowEditing": false } }` to `theme.json`.
- **Plugin/Block developers:** No direct API changes, but the `core/block-editor/toggle-block-visibility` shortcut will be unregistered if the theme opts out.
- **Site owners/Editors:** Visibility controls disappear from the editor UI when the active theme disables them, but existing visibility attributes remain in the block markup and persist across theme switches.
- **No migration required.** Existing sites and blocks are unaffected.

## Technical details

The change introduces a new global-only `theme.json` setting under `settings.blockVisibility.allowEditing` (default `true`). In `WP_Theme_JSON_Gutenberg`, the setting is added to the default schema and explicitly removed from per-block schemas to enforce global scope. The JSON schema (`schemas/json/theme.json`) registers `settingsBlockVisibilityProperties` and attaches it to `settingsAppearanceToolsProperties`.

On the editor side, components read the setting via `useSettings( 'blockVisibility.allowEditing' )` or `getSettings().__experimentalFeatures?.blockVisibility?.allowEditing`. When `false`:
- `BlockVisibilityViewportToolbar` and `BlockVisibilityViewportMenuItem` return `null`.
- `KeyboardShortcutsRegister` conditionally calls `unregisterShortcut( 'core/block-editor/toggle-block-visibility' )` to prevent a dead shortcut from lingering in the help modal.
- `BlockTools`, `ListViewBlock`, and `useQuickActionsCommands` short-circuit visibility-related handlers and commands.

Saved block attributes are untouched; the setting only gates UI rendering and shortcut registration.

## Contribution

Opened and merged by @ramonjd, with co-authors @tellthemachines, @andrewserong, and @carolinan. The PR evolved through discussion on whether visibility attributes should survive theme switches; the team aligned on preserving them to match existing block support behavior, opting to only hide the UI controls. The author noted pending retesting after a related breakpoint configuration PR (#79104) lands.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
