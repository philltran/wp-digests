# #82514: DataForm: render untyped fields without an edit control that are read-only

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dmallory42
- **Labels:** `[Type] Bug`, `First-time Contributor`, `[Package] DataViews`
- **Merged:** [`f16f9ed`](https://github.com/WordPress/gutenberg/commit/f16f9edb8c60179003c24e9dc8c0a4c224c4a0bf)
- **Discussion:** [#82514](https://github.com/WordPress/gutenberg/pull/82514) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

DataForm's `regular` and `card` layouts silently dropped any field that lacked an `Edit` component, even when the field was `readOnly: true` and defined its own `render`. The fix lets read-only fields render through their display renderer without needing `Edit` or a `type` that supplies one. Editable fields with no edit control remain hidden, as before.

## Impact

- **Plugin & theme developers using `@wordpress/dataviews`:**
  - Untyped, `Edit`-less fields with `readOnly: true` and a `render` function now appear in DataForm. Previously they were skipped.
  - Editable fields (no `readOnly`) without `Edit` or a type-provided `Edit` are still not rendered.
  - If you worked around the bug by adding a dummy `type` or `Edit` to information-only fields, that is no longer needed, though the maintainers still recommend setting a `type`.
- **Site owners / hosting / REST consumers:** No impact. No field API, saving behavior, or REST surface changes.
- **Breaking changes:** None. Forms that contain such fields will start showing them, which may be a visible change.

## Technical details

The diff adds a helper, `packages/dataviews/src/components/dataform-layouts/can-render-field.ts`:

```ts
export function canRenderField< Item >(
	field: NormalizedField< Item > | undefined
): field is NormalizedField< Item > {
	return !! field && ( field.readOnly === true || !! field.Edit );
}
```

It replaces the `! fieldDefinition || ! fieldDefinition.Edit` early-return guard in two layouts:

- `dataform-layouts/regular/index.tsx` (`FormRegularField`)
- `dataform-layouts/card/index.tsx` (`FormCardField`)

In the regular layout, both the `labelPosition: 'side'` branch and the top/none branch now render `fieldDefinition.Edit` only when it exists (`fieldDefinition.Edit && <fieldDefinition.Edit ... />`). Read-only fields continue to go through the existing render path. The label is omitted for `labelPosition: 'none'`.

Tests in `dataform.jsdom.test.tsx` cover:

- Read-only untyped fields in regular mode with `top`, `side` and `none` labels.
- An editable field without `Edit` staying hidden in regular mode.
- A read-only field rendering as its own card in card mode.
- An editable field without `Edit` not producing an empty `.dataforms-layouts-card__field` in card mode.

Storybook stories for the card, details, panel, regular and row layouts gain untyped read-only `file_type`/`fileType` fields. The `CHANGELOG.md` gets a Bug Fixes entry.

## Contribution

A first-time contributor, @dmallory42, opened this for a WooCommerce use case where information-only fields supplied a custom renderer but no `type`. @oandregal explained that the field type is not yet required but is strongly recommended and intended to become required. The author agreed to add `type: 'text'` in WooCommerce and argued the fix is still worthwhile while the API permits untyped fields. @oandregal added stories for each layout and approved, with a review comment that needed fixing before merge. The PR description discloses that Codex implemented the fix and tests.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
