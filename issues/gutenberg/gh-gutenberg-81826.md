# #81826: UI: Expose keyboard shortcut presentation utilities

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`a1e8d1d`](https://github.com/WordPress/gutenberg/commit/a1e8d1dfaabb12756ec8f10ccd4b9515ec02de7c)
- **Discussion:** [#81826](https://github.com/WordPress/gutenberg/pull/81826) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/ui` now exports three keyboard shortcut presentation/accessibility helpers: `KeyboardShortcutDescription`, `KeyboardShortcutDisplay`, and `useKeyboardShortcutProps`. They were previously internal, shared by `IconButton` and `Menu`. Consumers can now compose a tooltip hint, `aria-keyshortcuts`, and a visually hidden description on their own buttons without a wrapper taking over the button or tooltip. The Button story and the Boot save button are migrated to use them.

## Impact

**Plugin & theme developers (using `@wordpress/ui`)**
- New public exports available for adding shortcut hints and a11y metadata to any trigger element, e.g. a `Tooltip.Trigger` wrapping a `Button`.
- No breaking changes or deprecations. No action required unless you hand-roll this pattern (`useId` + `VisuallyHidden` + manual `aria-keyshortcuts`) and want to replace it.
- No public shortcut type or formatter is added; you build the `{ displayShortcut, ariaKeyShortcut, label }` object yourself, typically from `@wordpress/keycodes`.

**Tooling**
- The `@wordpress/eslint-plugin` `use-recommended-components` rule now allows these three names from `@wordpress/ui`, so importing them no longer triggers the "not yet recommended" report.

**Site owners:** No action required. The Boot save button's tooltip and shortcut metadata should behave the same.

## Technical details

**Exports:** `packages/ui/src/index.ts` adds a named re-export of `KeyboardShortcutDescription`, `KeyboardShortcutDisplay`, and `useKeyboardShortcutProps` from `./utils/keyboard-shortcut`.

**Component changes** (`packages/ui/src/utils/keyboard-shortcut.tsx`): `KeyboardShortcutDescription` and `KeyboardShortcutDisplay` are now wrapped in `forwardRef<HTMLSpanElement, …>`, forwarding the ref to the rendered `<span>`. `useKeyboardShortcutProps` gains doc comments, including a note to render `KeyboardShortcutDescription` with the returned `descriptionId` when a shortcut is provided. Behavior otherwise is unchanged: the display span is `aria-hidden="true"` with `dir="ltr"`, and the description is a `VisuallyHidden` span with `aria-hidden="true"`.

**ESLint:** `rules/use-recommended-components.js` adds the two components and the hook to the `@wordpress/ui` allowlist, with a new valid test case.

**Boot save button** (`packages/boot/src/components/save-button/index.tsx`): builds a `shortcut` object from `displayShortcut.primary('s')`, `ariaKeyShortcut.primary('s')`, and `shortcutAriaLabel.primary('s')`. It drops the hand-set `aria-keyshortcuts={ rawShortcut.primary('s') }` and spreads `targetProps` from `useKeyboardShortcutProps` on `Tooltip.Trigger`. The label and `KeyboardShortcutDescription` now render as children of `Tooltip.Trigger` (the `Button` is passed via `render` with no children), and the popup uses `KeyboardShortcutDisplay`. The `aria-keyshortcuts` value changes from `rawShortcut.primary('s')` to `ariaKeyShortcut.primary('s')`.

```tsx
const { descriptionId, targetProps } = useKeyboardShortcutProps( { shortcut } );

<Tooltip.Trigger { ...targetProps } render={ <Button /> }>
	{ label }
	{ descriptionId && (
		<KeyboardShortcutDescription descriptionId={ descriptionId } shortcut={ shortcut } />
	) }
</Tooltip.Trigger>
<Tooltip.Popup>
	<KeyboardShortcutDisplay shortcut={ shortcut } />
</Tooltip.Popup>
```

**Story and tests:** The Button `WithKeyboardShortcut` story now uses the helpers, merging consumer `aria-describedby` and `aria-keyshortcuts` through the hook. A new `keyboard-shortcut.test.tsx` covers ref forwarding, the accessible description, preservation of consumer `aria-describedby`/`aria-keyshortcuts`, and `aria-hidden`/`dir` on the display. CHANGELOGs are updated for `ui`, `boot`, and `eslint-plugin`. The bundle-size report shows `boot/index.min.js` up about 543 B.

## Contribution

This is a follow-up to #80321 and #80407 and closes #80321. The description notes that an aggregated formatter is deferred, to be explored in #80321 along with feedback from #81564 and the legacy key-value limitation raised in #74205. The PR states it deliberately avoids a public shortcut type or formatter. The author disclosed Codex assistance. Discussion on the PR is only bot output (size report, props list, one flaky e2e report).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
