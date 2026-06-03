# #78730: Post list: Remove close button from Quick Edit drawer

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @amitraj2203
- **Labels:** `[Type] Enhancement`, `[Package] Edit Site`, `[Feature] Site Editor`
- **Merged:** [`b6462a7`](https://github.com/WordPress/gutenberg/commit/b6462a7dc2b6018cc422c4ef98a5e793a1faf7d1)
- **Discussion:** [#78730](https://github.com/WordPress/gutenberg/pull/78730) · 2 comments · 0 reactions

## Summary

The X close button has been removed from the Quick Edit drawer in the Site Editor Pages DataView (Table layout). Previously the drawer offered two dismiss affordances — an X button in the `PostCardPanel` header and a **Cancel** button in the footer — creating ambiguity about whether closing via X would save or discard changes. The interaction model is now explicit: **Done** saves, **Cancel** discards and exits.

## Impact

**Plugin & theme developers**
- No breaking changes. `PostCardPanel` already gates the close button behind an optional `onClose` prop; this change simply stops passing that prop in the Quick Edit context. Any code that renders `PostCardPanel` and passes `onClose` is unaffected.

**Site owners / editors**
- The X button is gone from the Quick Edit drawer in Site Editor → Pages → Table layout. Users must use **Cancel** to discard or **Done** to save. No configuration change needed.

**Hosting & platform / headless consumers**
- No REST API, block schema, or database changes. No action required.

## Technical details

The effective change is a prop removal in two files:

- `packages/edit-site/src/components/post-list/quick-edit-modal.js`
- `routes/post-list/quick-edit-modal.tsx`

In both files, `PostCardPanel` was rendered with `onClose={ closeModal }`. Because `PostCardPanel` conditionally renders its header close button only when `onClose` is defined, removing the prop is sufficient to suppress the button — no changes to `PostCardPanel` itself were required.

**Before:**
```jsx
<PostCardPanel
  postType={ postType }
  postId={ postId }
  onClose={ closeModal }
/>
```

**After:**
```jsx
<PostCardPanel
  postType={ postType }
  postId={ postId }
/>
```

Bundle impact is −4 B on `build/scripts/edit-site/index.min.js`. The `AlertDialog` component in `@wordpress/components` is cited as the existing precedent for this "force an explicit action" pattern.

## Contribution

PR #78730 was opened and merged by @amitraj2203, resolving issue #78726. @jameskoster and @oandregal were credited as co-authors in the merge commit. The discussion thread was minimal (two bot comments); the design rationale — aligning with `AlertDialog`'s deliberate omission of a close button — was stated in the PR description without recorded dissent.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
