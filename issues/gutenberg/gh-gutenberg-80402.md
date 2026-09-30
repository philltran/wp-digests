# #80402: IconButton: Improve keyboard shortcut accessibility

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`07e90d6`](https://github.com/WordPress/gutenberg/commit/07e90d65825cd8fed78e5e2183f5268ad3821160)
- **Discussion:** [#80402](https://github.com/WordPress/gutenberg/pull/80402) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`IconButton` in `@wordpress/ui` now gives keyboard shortcuts an explicit accessible description, where before it only set `aria-keyshortcuts` and showed the shortcut in the tooltip. The `shortcut` prop now requires a plain-text `label`, which is rendered in a visually hidden element as "Keyboard shortcut: <label>" and linked to the button via `aria-describedby`. The visual shortcut in the tooltip stays `aria-hidden` and is now forced to `dir="ltr"`. Shared logic is extracted into internal helpers in `utils/keyboard-shortcut.tsx`.

## Impact

- **Plugin & theme developers using `@wordpress/ui` `IconButton` with `shortcut`:** **Breaking change.** `shortcut.label` is now required (TypeScript type error if omitted). Generate it with `shortcutAriaLabel` from `@wordpress/keycodes`, e.g. `shortcutAriaLabel.primary( 'c' )`.
- Existing `aria-describedby` and `aria-keyshortcuts` props passed to `IconButton` are preserved: the description ID is appended to `aria-describedby`, and `aria-keyshortcuts` is still honored when no `shortcut` is given.
- Shortcut registration and keeping the handler in sync with the disabled state remain the consumer's responsibility.
- **Site owners / headless / hosting:** No action required. This affects only the `@wordpress/ui` package, which is still pre-stable per the discussion.
- **Assistive technology users:** Screen readers get a readable shortcut description (e.g. "Keyboard shortcut: Command S") on the button.

## Technical details

**New file `packages/ui/src/utils/keyboard-shortcut.tsx`** exports:
- `KeyboardShortcut` type: `{ displayShortcut, ariaKeyShortcut, label }`. `label` is new and required.
- `useKeyboardShortcutProps( { 'aria-describedby', 'aria-keyshortcuts', shortcut } )`: calls `useId()`, and only when `shortcut` is set returns a `descriptionId`. It builds a space-joined `aria-describedby` from the consumer's value plus the generated ID, and sets `aria-keyshortcuts` to `shortcut.ariaKeyShortcut ?? ariaKeyShortcuts`.
- `KeyboardShortcutDescription`: a `VisuallyHidden` `<span>` with the `descriptionId` and `aria-hidden="true"`, containing `sprintf( __( 'Keyboard shortcut: %s' ), shortcut.label ?? shortcut.ariaKeyShortcut )`. It is hidden from the accessibility tree directly but still resolves as an `aria-describedby` target.
- `KeyboardShortcutDisplay`: `<span aria-hidden="true" dir="ltr">` showing `displayShortcut`.

**`icon-button.tsx`:** destructures `aria-describedby` and `aria-keyshortcuts` from props, spreads `targetProps` onto `Tooltip.Trigger`, renders `KeyboardShortcutDescription` as a child of the trigger, and uses `KeyboardShortcutDisplay` in the tooltip popup. The inline `aria-keyshortcuts={ shortcut?.ariaKeyShortcut }` on `Button` is removed.

**`types.ts`:** the inline `shortcut` object type is replaced by the shared `KeyboardShortcut`. The docs now say the consumer must keep the handler synchronized with the button's disabled state.

```tsx
// Before
shortcut={ {
	displayShortcut: displayShortcut.primary( 'c' ),
	ariaKeyShortcut: ariaKeyShortcut.primary( 'c' ),
} }

// After
shortcut={ {
	displayShortcut: displayShortcut.primary( 'c' ),
	ariaKeyShortcut: ariaKeyShortcut.primary( 'c' ),
	label: shortcutAriaLabel.primary( 'c' ),
} }
```

Tests cover the merged accessible description alongside an external `aria-describedby`, direct ARIA prop preservation without `shortcut`, `dir="ltr"`, and metadata being kept when the button is focusable while disabled. The `CHANGELOG.md` records this under Breaking Changes. The `?? shortcut.ariaKeyShortcut` fallback in the description is a runtime guard for JS callers that omit `label`.

## Contribution

This is a follow-up to #80321 (see also #80353, #79560). @mirka reviewed and, after @ciampo addressed feedback, remarked this may be the last step before marking `IconButton` (and probably `Button`) as officially recommended. @ciampo noted a separate PR (#80407) adding a shortcut-focused high-level `Button`, which is not a blocker for `Button` stabilization. The PR description says it was implemented with AI assistance in Codex and reviewed locally.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
