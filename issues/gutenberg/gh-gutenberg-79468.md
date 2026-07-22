# #79468: Media Fields: Avoid focus loss when detaching the current parent

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Feature] Media`
- **Merged:** [`8604481`](https://github.com/WordPress/gutenberg/commit/86044811ecc18bd36ae4aaceb6f3a5e66d3304c0)
- **Discussion:** [#79468](https://github.com/WordPress/gutenberg/pull/79468) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request fixes an accessibility regression in the Media Fields block where clicking "detach current" caused focus loss by conditionally hiding the action button. The fix keeps the button in the DOM but disables it when no media is attached, preserving keyboard navigation flow. This ensures screen reader and keyboard users retain focus during the detach action.

## Impact

- **Block & plugin developers**: No code changes required; the fix is internal to the `@wordpress/media-fields` package.
- **Site owners & editors**: Improved keyboard and screen reader experience when detaching media from posts/pages in the block editor.
- **No breaking changes, deprecations, or migration steps.**

## Technical details

The change modifies `packages/media-fields/src/attached_to/edit.tsx`. Previously, the help text conditionally rendered a `<Button>` with `variant="link"` only when `data.post` was truthy. Clicking it triggered `handleDetach`, which cleared state and hid the button, causing focus loss. The diff updates the component to always render the button and applies `disabled={ ! value }` to prevent interaction when unattached. It also explicitly calls `setValue( null );` before `setOptions( [] );` in the detach handler to ensure state resets cleanly. The `ComboboxControl` wrapper and `accessibleWhenDisabled` prop remain unchanged.

## Contribution

Opened by @Mamaduka as a follow-up to #74432, the PR addressed a focus-loss regression introduced when hiding the detach action. During review, @Mamaduka proposed using `ComboboxControl`'s built-in reset action instead, but @andrewserong and @ramonjd preferred retaining the explicit link-style button for clarity. The team agreed the disabled-state approach best balanced accessibility with the current MVP UI, and the change was merged without further modification.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
