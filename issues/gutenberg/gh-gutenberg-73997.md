# #73997: Pattern editing: fade blocks outside the edited pattern in List View

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Patterns`
- **Merged:** [`c605983`](https://github.com/WordPress/gutenberg/commit/c6059838653e53268d6686ee2b269243c7e3d6fd)
- **Discussion:** [#73997](https://github.com/WordPress/gutenberg/pull/73997) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

When editing a content-only pattern in the block editor, blocks outside the active pattern are now visually faded and disabled in List View. This enforces interaction boundaries that match canvas mode, preventing accidental focus traversal or selection outside the pattern until explicitly exited via the Escape key or by clicking an external block.

## Impact

- **Editors & pattern authors:** Will experience a constrained navigation zone while editing patterns; out-of-pattern List View items appear muted and cannot be tabbed into or clicked unless explicitly triggered.
- **Theme & plugin developers:** No code changes required. The internal selector responsible for filtering the visible block tree has been refactored, but remains private to the `@wordpress/block-editor` package.
- **No action required** for end users or standard theme/plugin maintainers.

## Technical details

The change consolidates List View visibility logic into a new `getListViewClientIdsTree` selector (exported in `packages/block-editor/src/store/private-selectors.js`). This selector replaces the previous `getEnabledClientIdsTree` call in `packages/block-editor/src/components/list-view/use-list-view-client-ids.js` and uses `getFilteredClientIdsTreeUnmemoized` to evaluate `blockEditingMode === 'disabled'` against `state.editedContentOnlySection`. 

In `@wordpress/block-editor/components/list-view/block.js`, disabled blocks now receive an `isDisabled` prop that strips hover/focus event handlers, forces `tabIndex={-1}`, and conditionally omits the `href` attribute while setting `aria-disabled={true}`. The `.is-disabled` modifier in `style.scss` applies `opacity: 0.2` with a 0.1s linear transition and neutralizes cursor/color hover states. Additionally, `edit-contents.js` now explicitly calls `selectBlock(clientId)` when toggling the inline edit button to maintain expected selection state.

## Contribution

Opened by @ramonjd to resolve #73954, the PR was iteratively refined by @talldan and @tellthemachines. Key revisions included removing an experiment flag, decoupling the faded context behavior into its own selector to avoid scope leakage, and locking down keyboard navigation so that disabled items are properly skipped via roving tabindex while Escape reliably exits pattern editing. Merged after passing e2e coverage and receiving positive UX validation.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
