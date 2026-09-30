# #82632: Keycodes: Add a shared keyboard shortcut representation helper

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Keycodes`, `[Package] Editor`, `[Package] Edit Post`, `[Package] Keyboard Shortcuts`
- **Merged:** [`70adb5e`](https://github.com/WordPress/gutenberg/commit/70adb5ecde36daa12ed0295bda04be9c235496bf)
- **Discussion:** [#82632](https://github.com/WordPress/gutenberg/pull/82632) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a `keyboardShortcut` helper to `@wordpress/keycodes` and a `getKeyboardShortcut` selector to the `@wordpress/keyboard-shortcuts` store. Each returns `{ displayShortcut, ariaKeyShortcut, label }` in a single call, which is the object shape `@wordpress/ui` components such as `Menu.Item` and `IconButton` take for their `shortcut` prop. The editor's private `getKeyboardShortcut` util is removed, and the editor more menu, mode switcher and edit-post fullscreen item now use the new helpers. There is no user-visible change.

## Impact

- **Plugin & theme developers:** Two new public APIs are available for building `shortcut` props for `@wordpress/ui` components: `keyboardShortcut.<modifier>( character )` for literal shortcuts, and `select( keyboardShortcutsStore ).getKeyboardShortcut( name )` for registered ones. Neither replaces existing APIs. `displayShortcut`, `ariaKeyShortcut`, `shortcutAriaLabel` and `getShortcutRepresentation` are unchanged.
- **Compatibility:** The new symbols only exist in package versions that include this change. Code that runs against the WordPress-bundled copies of these packages on older WordPress versions should not assume they are present. Reviewers raised this for the bundled `boot` and `ui` packages; the PR left the bundled packages' original code in place and documents the new methods for other consumers to adopt.
- **Public contract:** The `{ displayShortcut, ariaKeyShortcut, label }` shape and its string types are now public API that Gutenberg will need to keep supporting.
- **Site owners / hosting / REST consumers:** No action required.

## Technical details

**`@wordpress/keycodes`** (`packages/keycodes/src/index.ts`)
- New exported type `WPKeyboardShortcut = { displayShortcut: string; ariaKeyShortcut: string; label: string }`.
- New export `keyboardShortcut`, typed `WPModifierHandler<WPKeyHandler<WPKeyboardShortcut>>`. It is built with `Object.fromEntries` over `Object.keys( modifiers )`. Each modifier function calls `displayShortcut[ modifier ]`, `ariaKeyShortcut[ modifier ]` and `shortcutAriaLabel[ modifier ]` with the same `character` and `_isApple` (default `isAppleOS`). It is marked `/* @__PURE__ */`.
- The no-modifier case is exposed as `keyboardShortcut.undefined( 'm' )`.
- Tests in `index.jsdom.test.ts` cover Windows, macOS and the no-modifier case.

```js
// Assuming macOS:
keyboardShortcut.primaryShift( 'm' );
// { displayShortcut: '⇧⌘M', ariaKeyShortcut: 'Shift+Meta+M', label: 'Shift Command M' }
```

**`@wordpress/keyboard-shortcuts`** (`packages/keyboard-shortcuts/src/store/selectors.ts`)
- New selector `getKeyboardShortcut( state, name )`, built with `createSelector` and dependent on `state[ name ]`. It resolves `getShortcutKeyCombination( state, name )`, returns `null` if nothing is registered, and otherwise returns `keyboardShortcut[ combination.modifier ?? 'undefined' ]( combination.character )`.
- Only the main key combination is covered. Raw representation and aliases are not included; use `getShortcutRepresentation` and `getShortcutAliases` for those.
- No selector tests were added. The author judged formatting already covered in keycodes and the selector to add only a `null` check over `createSelector`.

**Call-site migrations**
- `editor/src/components/more-menu/index.jsx`: the module-level constants from the private util are replaced with `keyboardShortcut.primaryShift( '\\' )` and `keyboardShortcut.access( 'h' )`.
- `editor/src/components/mode-switcher/index.jsx`: now calls `getKeyboardShortcut( 'core/editor/toggle-mode' )` via `useSelect`, replacing `getShortcutKeyCombination` plus the util.
- `edit-post/src/components/more-menu/index.jsx`: the `FULLSCREEN_SHORTCUT` constant is replaced with `keyboardShortcut.secondary( 'f' )`.
- `packages/editor/src/utils/keyboard-shortcut.js` is deleted.

**Docs:** CHANGELOG entries were added for both packages, plus a README section for keycodes and the generated data reference for the selector.

## Contribution

@Mamaduka proposed the change as a follow-up to #81564, to stop call sites from each hand-rolling the `@wordpress/ui` shortcut object. @mirka questioned the design, saying the helper has a narrow audience and ties a `@wordpress/ui` prop shape to a global package under strict back-compat. She suggested adding an `ariaKeyShortcut` representation to the existing `getShortcutRepresentation` instead. Mamaduka chose a new selector to avoid changing the existing selector's return value. @ciampo backed the approach. He noted the main commitment is the public shape, called the `getShortcutRepresentation` alternative smaller but not helpful for literal shortcut call sites, and asked for clearer docs and a corrected example. Mamaduka made those doc fixes but declined to add selector tests. Late review also flagged older-WordPress compatibility for bundled packages, so Mamaduka restored the original code in the bundled `boot`/`ui` packages.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
