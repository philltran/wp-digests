# #82844: Fields: Hide the date, author and password fields when the user can't publish or reassign

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `[Feature] DataViews`, `[Package] Fields`
- **Merged:** [`8a1e63d`](https://github.com/WordPress/gutenberg/commit/8a1e63d0d0b70718c747535427dcc718f3c75805)
- **Discussion:** [#82844](https://github.com/WordPress/gutenberg/pull/82844) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/fields` date, author and password fields now hide themselves when the post's REST `_links` lack the matching `wp:action-publish` or `wp:action-assign-author` relation. Previously the DataForm-based post summary showed these controls to contributors, who could not use them: saving as Published failed with `rest_cannot_publish`, and picking another author failed too. The behavior now matches the classic post summary sidebar.

## Impact

- **Plugin & theme developers using `@wordpress/fields` / DataForm:** Date, author and password fields return `isVisible: false` for records whose `_links` exist but omit the relevant action relation. If you render these fields for restricted users and relied on them always appearing, expect them to disappear.
- **Contributors / limited roles (Editor Inspector: Use DataForm experiment):** The summary no longer shows `Date` and `Author` rows, and the `Edit Status` popover no longer shows `Password protected`. Authors (can publish, can't reassign) keep `Date` but lose `Author`.
- **Bulk quick edit:** Unaffected. A record with no `_links` counts as allowed, so `Date` and `Author` still appear for admins in the pages list bulk form.
- **Known gaps (follow-ups per the PR):** `status` and `slug` remain editable for contributors, and an empty `Template` row still shows for non-admins.
- No migration or code changes required.

## Technical details

The diff adds `packages/fields/src/fields/utils.ts` exporting `hasActionLink( item: BasePost, action: string ): boolean`. It returns `true` when `item._links` is absent, otherwise `!! item._links[ action ]`.

Field changes:

- `date`: new `isVisible: ( item ) => hasActionLink( item, 'wp:action-publish' )`
- `author`: new `isVisible: ( item ) => hasActionLink( item, 'wp:action-assign-author' )`
- `password`: `isVisible` goes from `item.status !== 'private'` to `item.status !== 'private' && hasActionLink( item, 'wp:action-publish' )`
- `sticky`: refactored from `!! item._links?.[ 'wp:action-sticky' ]` to `hasActionLink( item, 'wp:action-sticky' )`

The sticky refactor also changes behavior slightly: a record without `_links` was previously hidden and is now visible.

```ts
// before (password)
isVisible: ( item ) => item.status !== 'private',
// after
isVisible: ( item ) =>
	item.status !== 'private' && hasActionLink( item, 'wp:action-publish' ),
```

No new hooks, REST schema changes or DB changes. Added e2e coverage: a contributor-login test in `post-summary.spec.js` asserting Date, Author and Password are hidden, and a bulk Quick Edit test in `page-list.spec.js` asserting Status, Date, Author and Discussion still show. A CHANGELOG entry was added under Bug Fixes.

## Contribution

Opened by @ntsekouras as part of the tracking issue #76076 and merged after a short review. @oandregal asked why contributors would see a password control at all; the reply was that contributors can't actually publish, but the UI offered the control, and that making `status` read-only in the DataForm experiment is a planned follow-up. The classic inspector makes status read-only, so it shows no popover. The change was kept because view config filters can place the password field outside the `status` group.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
