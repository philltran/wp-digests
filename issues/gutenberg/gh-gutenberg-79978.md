# #79978: Media Inserter: Guard attach, detach, and invalidate behind a ! isExternalResource check

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Feature] Media`, `[Package] Block editor`
- **Merged:** [`68f7097`](https://github.com/WordPress/gutenberg/commit/68f709736c3eb730c333ed0c0cb1efa0bd9bfc7e)
- **Discussion:** [#79978](https://github.com/WordPress/gutenberg/pull/79978) · 5 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

This change hardens the block editor's media inserter by gating the `attach`, `detach`, and `invalidate` methods behind an `isExternalResource` flag. Previously, any custom media category registering these callbacks could inadvertently trigger the core Attachments workflow. The fix ensures that attachment-based operations are exclusively applied to WordPress's built-in categories, preserving external resource sources from unintended side effects or API collisions.

## Impact

['- Plugin/theme developers extending the media inserter: No action required. External resource categories (`isExternalResource: true`) will continue to function correctly without accidentally triggering core attachment workflows.', "- Core block editor consumers: The `registerInserterMediaCategory` action's contract is reinforced, clarifying that `isExternalResource` prevents unnecessary fetch checks when displaying media tabs.", '- No breaking changes or deprecations introduced; this is a behavioral safeguard for internal media source registration.']

## Technical details

Modifies `MediaCategoryPanel` in `packages/block-editor/src/components/inserter/media-tab/media-panel.js` to conditionally expose attachment workflow methods. It computes `supportsAttachments = !category.isExternalResource` and assigns local variables (`const attach = supportsAttachments ? category.attach : undefined;`, same for `detach`) before passing them into `useCallback` hooks and JSX render props. This prevents third-party categories from opt-in-ing to the core Attachments lifecycle just by defining these props.

Before:
```javascript
await category.attach( selectedMedia );
const attachedCount = await category.attach( selectedMedia );
'has-attach-footer': !! category.attach,
onDetach={ category.detach ? setMediaPendingDetach : undefined }
```

After:
```javascript
const supportsAttachments = !category.isExternalResource;
const attach = supportsAttachments ? category.attach : undefined;
const detach = supportsAttachments ? category.detach : undefined;
await attach?.( selectedMedia );
'has-attach-footer': !! attach,
onDetach={ detach ? setMediaPendingDetach : undefined }
```
The PR also updates JSDoc in `packages/block-editor/src/store/actions.js` and `docs/reference-guides/data/data-core-block-editor.md` to clarify that `isExternalResource` should prevent unnecessary fetch requests when checking for media items, and adds a unit test file (`test/media-panel.js`) to verify the gating logic via mocked render props.

## Contribution

Opened by @andrewserong as a follow-up to PR #79336, with review feedback from @ntsekouras noting that the methods were technically exposed despite intended private usage. The team settled on guarding these behaviors behind the existing `isExternalResource` flag rather than hiding the props entirely, ensuring a clear, enforced contract for all registered media categories. @ramonjd co-authored and merged the change in commit `68f7097`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
