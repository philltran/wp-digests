# #78653: Media Editor: make the modal the default crop experience

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Feature] Media`, `[Package] Editor`
- **Merged:** [`be0395e`](https://github.com/WordPress/gutenberg/commit/be0395e7f8ad41df43fbc13b2fd9373be84acaeb)
- **Discussion:** [#78653](https://github.com/WordPress/gutenberg/pull/78653) · 12 comments · 4 reactions

## Summary

The modal-based Media Editor crop interface, previously opt-in via the `gutenberg-media-editor-modal` Gutenberg experiment, is now the unconditional default for all image editing in the block editor. The change removes the PHP experiment registration, the PHP bridge that injected `window.__experimentalMediaEditorModal`, and the JavaScript conditional in the editor provider that gated modal mounting. Clicking the Crop toolbar button on the Image block or Site Logo block now always opens the Media Editor modal. The inline `__experimentalImageEditor` component is deliberately left in the source tree as unreachable dead code; its removal is tracked in follow-up PR #78654.

## Impact

**Plugin & theme developers**
- If your code branches on `settings.openMediaEditorModal` being falsy to fall back to the inline editor, that branch is now unreachable — `openMediaEditorModal` is always a defined callback.
- The inline `__experimentalImageEditor` component remains in the tree for now but cannot be reached via normal editor flow; do not depend on it. Its removal is deferred to #78654.
- No consumer-side code changes are required to adopt this PR's behaviour.

**Site owners / content editors**
- The Media Editor modal is now the crop UX for Image and Site Logo blocks for all users, with no opt-in required.

**Hosting & platform**
- The `gutenberg-media-editor-modal` entry is removed from `wp-admin > Gutenberg > Experiments`. Any tooling, documentation, or feature-flag checks referencing that experiment name or `window.__experimentalMediaEditorModal` should be treated as obsolete.

## Technical details

Three files are modified (no unified diff was provided, but the PR describes each change precisely):

1. **`lib/experimental/experiments/load.php`** — the `gutenberg-media-editor-modal` experiment definition is removed entirely.
2. **`lib/experimental/editor-settings.php`** — the PHP bridge that set `window.__experimentalMediaEditorModal` on the page is removed.
3. **`packages/editor/src/components/provider/`** — the JS conditional that gated modal mounting is removed; the `openMediaEditorModal` block-editor setting is now supplied unconditionally.

The Image block and Site Logo block each contain a toolbar branch resembling:

```js
// Before (experiment gate in provider)
openMediaEditorModal ? openMediaEditorModal( clientId ) : /* inline __experimentalImageEditor */

// After (openMediaEditorModal is always defined)
openMediaEditorModal( clientId ) // modal path always taken
```

Neither consumer block was touched in this PR. The branch evaluates to the modal path because the callback is now always present.

The only changed build artifact is `build/scripts/editor/index.min.js` (+38 B), consistent with a small guard removal in the provider. No REST schema, `block.json` fields, or database changes are involved.

## Contribution

PR opened and merged by @ramonjd (commit `be0395e`). The modal had been opt-in since PR #77480; a Make WordPress Core call-for-testing post was published on 2026-05-21 ahead of flipping the default. The PR is explicitly scoped to the minimum change required — consumer block code and the inline editor component are intentionally left untouched. A companion draft, PR #78654, tracks the subsequent cleanup. Additional contributors credited: @andrewserong, @talldan, @jasmussen, @jameskoster.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
