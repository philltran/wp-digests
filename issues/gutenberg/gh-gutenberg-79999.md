# #79999: Restore responsive editing viewport dropdown copy changes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] Style States`
- **Merged:** [`5e18bf8`](https://github.com/WordPress/gutenberg/commit/5e18bf87245b79f30931996562a71c3c3efed737)
- **Discussion:** [#79999](https://github.com/WordPress/gutenberg/pull/79999) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Restores user-facing string translations in the block editor's responsive editing viewport dropdown that were inadvertently reverted due to a merge conflict resolution in #79104. The change updates three UI tooltips to explicitly reference "style changes" and "viewport", ensuring terminology accurately reflects the block styling context.

## Impact

- **Block editors & theme/plugin developers**: No functional or API changes. Developers using the editor's responsive dropdown will see clarified copy, but no code adjustments are needed.
- **Translation maintainers**: Three translatable strings in `packages/editor/src/components/preview-dropdown/index.js` have been modified. POT exports and translation repositories require updates to reflect the new wording.
- **No action required** for runtime configuration or backward compatibility.

## Technical details

The unified diff modifies `packages/editor/src/components/preview-dropdown/index.js` inside the `PreviewDropdown` component. It replaces three translatable strings tied to the `isResponsiveEditing` conditional:
- Tablet info: `'Make tablet exclusive changes.'` → `'Make tablet exclusive style changes.'`
- Mobile info: `'Make mobile exclusive changes.'` → `'Make mobile exclusive style changes.'`
- Toggle info: `'Edits apply only to the current state.'` → `'Style changes apply only to the current viewport.'`
These strings render via `__()` within the dropdown menu logic. The patch touches only string literals and does not alter component props, state management, or the underlying responsive editing mechanism.

## Contribution

Opened by @talldan after identifying that PR #79104 accidentally reverted copy updates originally introduced in #79615. Co-authored by @t-hamano and @ramonjd based on final commit metadata. @ramonjd provided a swift approval, noting it was a straightforward regression fix. Merged without design debate as the change strictly reverts text to its intended state.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
