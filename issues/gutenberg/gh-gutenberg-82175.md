# #82175: DataForm: treat combined form fields purely as layout containers

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @oandregal
- **Labels:** `[Type] Bug`, `[Package] DataViews`
- **Merged:** [`7b888e6`](https://github.com/WordPress/gutenberg/commit/7b888e67a46dc185d5d4057367d869a66369670f)
- **Discussion:** [#82175](https://github.com/WordPress/gutenberg/pull/82175) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

In `@wordpress/dataviews`, a combined DataForm field (a form field with `children`) is now treated purely as a layout container. Its `id` is no longer resolved against the field definitions, either in validation (`useFormValidity`) or in the `panel` layout's summary and `readOnly` lookup. Previously validation and panel rendering could pick up a same-id field definition for a group, while other layouts ignored it, so behavior was inconsistent. The PR is labelled a bug fix, but the CHANGELOG lists it under Breaking Changes.

## Impact

**Plugin/theme developers using DataForm / DataViews**
- **Breaking (documented in the package CHANGELOG):** if a combined field shared its `id` with a field definition to supply the panel summary, the group is now summarized by its first leaf child. Declare the summary explicitly with `layout.summary`.
- If a same-id field definition carried `isValid` rules that were applied to the group, those rules no longer run on the group. Move them to the child fields.
- A same-id definition's `isVisible` no longer affects the group either. Only children's visibility and validity count.
- Forms whose group ids do not overlap with field ids, or that already use `layout.summary`, are unaffected.

**Gutenberg / core**
- The default post type form's `status` and `discussion` groups now get an explicit `layout.summary` through a new PHP filter layer in `lib/compat/wordpress-7.2/view-config-api.php`. A core backport is tracked at `wordpress-develop#13308`.
- The QuickEdit modal and the panel stories were updated to set `layout.summary` explicitly.

**Site owners:** No action required.

## Technical details

**Validation** (`packages/dataviews/src/hooks/use-form-validity.ts`): `getFormFieldsToValidate` no longer calls `fieldsMap.get( formField.id )` for a combined field, and the branch that attached a normalized `field` to the group is removed. It now returns only `{ id, children }`, so only the children are validated. A new jsdom test covers a group `id` that matches a definition with `isValid: { required: true }` and `isVisible: () => false`, and asserts `validity` is `undefined` and `isValid` is `true`.

**Panel layout** (`panel/utils/use-field-from-form-field.ts`): `getFieldDefinition` now branches first on `field.children`. For a combined field it takes the first child without its own `children` and looks up that child's definition, or returns `undefined` if none exists. Leaf fields still resolve by their own `id`. `layout.summary` keeps priority, and the doc comment on the priority order was updated.

**Migration pattern:**

```js
// Before: summary came from the `discussion` field because the group shares its id
{ id: 'discussion', children: [ 'comment_status', 'ping_status' ] }

// After
{
	id: 'discussion',
	layout: { type: 'panel', summary: 'discussion' },
	children: [ 'comment_status', 'ping_status' ],
}
```

**Gutenberg PHP compat layer:** adds `_gutenberg_add_group_summaries_to_default_posttype_form()`, which merges `layout.summary` into the `status` and `discussion` groups by member id. `gutenberg_register_default_posttype_form_summaries_7_2()`, hooked on `registered_post_type`, attaches it at priority 6 on the `gutenberg_get_entity_view_config_hook_name( 'postType', $post_type )` filter. This covers custom post types. Post types listed in the new constant `GUTENBERG_VIEW_CONFIG_POST_TYPES_WITH_OWN_FORM` (`wp_block`, `wp_template`, `wp_template_part`) are skipped, because the merge would otherwise append missing members. PHPUnit tests cover `page`, `post`, a custom post type, and the excluded types.

**Other changes:** `routes/post-list/quick-edit-modal.tsx` and the `layout-panel.tsx` story add explicit `layout.summary` values for `status`, `discussion` and `address1`. The `page-list.spec.js` e2e test now clicks the "Upload files" tab before selecting a file. A backport-changelog entry was added under `backport-changelog/7.2/`.

## Contribution

The change follows a review thread on #81377, which asked what should happen when a combined field shares an id with a hidden field definition, for example whether the whole group should become invisible. @oandregal opted to make groups pure layout containers instead. He noted the opposite approach, matching groups to fields for every aspect, would also work but blurs the boundary. The stated trade-off is that group-level validation or visibility would need field API work, such as object or array fields. He held off merging to gather feedback and check other consumers of the library. A reviewer-reported bug in the summary handling was fixed by setting the `discussion` summary field explicitly. A companion `wordpress-develop` backport PR was opened for review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
