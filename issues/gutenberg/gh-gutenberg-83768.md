# #83768: Notes: Add show and hide options for floating notes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] Notes`
- **Merged:** [`8027397`](https://github.com/WordPress/gutenberg/commit/8027397e0b685d6cf0e48d60080596299a682539)
- **Discussion:** [#83768](https://github.com/WordPress/gutenberg/pull/83768) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg editor's Options menu gets a new "Notes" submenu in the View group, above Panels, with "Show notes" / "Hide notes" radio items and a "Show all notes" checkbox. Floating notes can now be hidden, and the choice persists in the `core` preference `notesDisplayMode`. Previously, closing any other sidebar brought floating notes back. The "All notes" sidebar no longer appears as an item in the Panels menu; the new submenu toggles it instead.

## Impact

**Site owners / editors**
- Can hide floating notes persistently (across sidebar open/close and reloads). Adding a note, or opening a note via the existing flow while hidden, sets the mode back to shown.
- "All notes" now lives under Options > Notes instead of Options > Panels. The PR description calls it "All notes"; the merged UI label is "Show all notes" (per the diff and e2e test).
- The Notes submenu is hidden on small viewports unless the "All notes" sidebar is available; on those viewports only the "Show all notes" checkbox appears.

**Plugin & theme developers**
- No action required for most. No new public hook or registered API; `NotesMoreMenuGroup` is an internal slot/fill within `@wordpress/editor`, not exported in this diff.
- Anything relying on the "All notes" `PluginSidebar` appearing in the Panels menu (e.g. e2e tests or docs) will no longer find it there, because the `name` prop was removed.
- The `notesDisplayMode` preference (`'full'` | `'hidden'`; unset behaves as `'full'`) is readable via `select( 'core/preferences' ).get( 'core', 'notesDisplayMode' )`.

## Technical details

**New files**
- `packages/editor/src/components/collab-sidebar/notes-display-mode-menu.tsx`: `NotesDisplayModeMenu` renders into a `NotesMoreMenuGroup.Fill` a `MoreMenuSubmenu` labelled "Notes". It uses `Menu.RadioGroup` (values `full` / `hidden`) from `@wordpress/ui` and a `Menu.CheckboxItem` for the "All notes" sidebar. `setDisplayMode` writes the preference via `preferencesStore`'s `set`, calls `enableComplementaryArea( 'core', FLOATING_NOTES_SIDEBAR )` when switching to `full` with floating notes present (replacing any open sidebar), and announces via `speak()` ("Notes hidden." / "Notes shown."). The radio group is only rendered on medium+ viewports (`useViewportMatch( 'medium' )`).
- `packages/editor/src/components/more-menu/notes-more-menu-group.tsx`: `createSlotFill( Symbol( 'NotesMoreMenuGroup' ) )`.

**Changed files**
- `more-menu/index.jsx`: renders `<NotesMoreMenuGroup.Slot />` right after the View submenu (`ViewMoreMenuGroup.Slot`).
- `collab-sidebar/index.jsx`:
  - Reads `areNotesHidden` from `get( 'core', 'notesDisplayMode' ) === 'hidden'`.
  - Extracts `hasFloatingNotes` and calls `useEnableFloatingSidebar( hasFloatingNotes && ! areNotesHidden )`.
  - In `focusNote`, when the floating sidebar is in use and notes are hidden, sets the preference back to `'full'` before enabling the complementary area.
  - Removes `name={ ALL_NOTES_SIDEBAR }` from the "All notes" `PluginSidebar`, which drops the automatic Panels entry while keeping its pinned toolbar button (per the PR description).
- `collab-sidebar/hooks.js`: only a comment is added in the diff, noting that hiding the complementary area only changes the preferences store. The PR description says `useEnableFloatingSidebar` now subscribes to `preferencesStore` only, but that code change is not visible in the provided diff, so treat it as unverified.
- `CHANGELOG.md` entry under `@wordpress/editor` enhancements; e2e coverage added in `block-notes-floating.spec.js` plus a `clickNotesMenuItem` helper in `block-notes-utils.js`.

```js
// Preference semantics
get( 'core', 'notesDisplayMode' ) === 'hidden' // hidden
// anything else (unset or 'full') => floating notes shown
```

## Contribution

@Mamaduka opened this as a smaller, standalone alternative to #76024 (display modes including a minimize mode), which is stacked on the canvas-margin work in #79864. @jasmussen approved the approach and suggested the label "Show all notes" (which the merged code uses) and noted a possible follow-up with three modes: expand, minimize, hide. @Mamaduka kept "All notes" as an archive, since floating notes only display unresolved ones, and deliberately left out "Minimize" because it doesn't work unless the canvas can shrink. @annezazu tested it briefly, and it was merged as a foundation for that later work.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
