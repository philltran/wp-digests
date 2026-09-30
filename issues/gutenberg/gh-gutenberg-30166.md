# #30166: Button: Only open the link popover on explicit user action

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @stokesman
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Buttons`
- **Merged:** [`ba319c8`](https://github.com/WordPress/gutenberg/commit/ba319c8a879cc45fe3b08b8fca09cc66947089d3)
- **Discussion:** [#30166](https://github.com/WordPress/gutenberg/pull/30166) · 16 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Button block's link popover no longer opens automatically when a linked button is selected. It now opens only when the user clicks the "Link" toolbar button or presses Cmd/Ctrl+K. The toolbar always shows "Link" instead of swapping to "Unlink" once a URL is set, and the button appears pressed when a link exists. This also fixes the popover staying visible while editing as HTML or in Select (Navigation) mode.

## Impact

- **Editors / site owners:**
  - Selecting a linked button no longer covers surrounding content with the popover.
  - The "Unlink" toolbar button is gone. Links are removed with Cmd/Ctrl+Shift+K or the remove button inside the popover.
  - The Link button toggles the popover open and closed, and shows a pressed state when a URL is set.
- **Plugin / theme developers:**
  - There is no API change and no attribute or serialization change.
  - Custom e2e or Playwright tests that expect the popover to appear on selecting a linked button will break. Core's own specs had to add an explicit `primary+k` press. The `buttons.spec.js` assertion that the link control is visible after a URL is set was removed.
  - Tests that look for an "Unlink" toolbar button must change.
- **Headless / REST / hosting:** No impact.
- No migration is required.

## Technical details

The change is in `packages/block-library/src/button/edit.jsx`, based on the diff in the discussion.

- **Popover condition.** The `Popover` containing `LinkControl` was previously rendered when `isLinkTag && isSelected && (isEditingURL || isURLSet) && !lockUrlControls`. It now renders when `isLinkTag && isSelected && isEditingURL && !lockUrlControls`. `isURLSet` no longer triggers it. The reported bug was that `onClose` set `isEditingURL` to false but `isURLSet` kept the popover open.
- **Toolbar button.** The `ToolbarButton` is no longer conditional on `isURLSet`. It always uses the `link` icon, the title `Link` and the `displayShortcut.primary( 'k' )` shortcut. Its `onClick` is `() => setIsEditingURL( ! isEditingURL )`, and `isActive={ isURLSet }`. The `linkOff` import is removed.
- **`LinkControl`.** `forceIsEditingLink` changed from `isEditingURL` to `! isURLSet`. With no link, the popover opens with the URL input ready. With a link, it shows the preview first.
- **Unchanged.** The keydown handling for Cmd/Ctrl+K and Cmd/Ctrl+Shift+K, `onRemove`, `onClose` refocusing the rich text, and the `createSuggestion` props are unchanged.

```jsx
// before
icon={ ! isURLSet ? link : linkOff }
onClick={ ! isURLSet ? startEditing : unlink }
// after
icon={ link }
onClick={ () => setIsEditingURL( ! isEditingURL ) }
isActive={ isURLSet }
```

The diff also adds a CHANGELOG entry and updates e2e specs: `buttons.spec.js`, `block-bindings/custom-sources.spec.js` and `pattern-overrides.spec.js` now press `primary+k` to open the popover. The bot reports a `build/block-library/index.min.js` size change of +45 B.

## Contribution

@stokesman opened this in March 2021 to hide the popover in HTML-edit and Select modes, and it sat open for about five years. @talldan first asked how to reproduce the bug. @t-hamano then diagnosed the root cause: `onClose` reset `isEditingURL`, but `isSelected && isURLSet` kept the popover open. Rather than gating the popover on edit mode, he proposed matching the Image block's link UX: no auto-open, an always-present toggling "Link" button, and no unlink on that button. @stokesman agreed and said he preferred that approach, having assumed the auto-open was intentional. The final revision took @t-hamano's approach, with Claude Code used to resolve conflicts, update e2e shortcuts and draft the description.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
