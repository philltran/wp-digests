# #78825: DataViews: add a richtext dataform control and migrate fields to it

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Feature`, `[Package] DataViews`, `[Package] Fields`
- **Merged:** [`6ab2e4b`](https://github.com/WordPress/gutenberg/commit/6ab2e4b1fce0165636e0f75ab172c953f6e86833)
- **Discussion:** [#78825](https://github.com/WordPress/gutenberg/pull/78825) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The DataViews package now ships a built-in `richtext` dataform control, allowing any DataViews consumer to render rich-text fields without importing `RichTextControl` directly. This change migrates the core `rich-text` and `title` fields in `@wordpress/fields` to use the new control, centralizing the wrapper and removing a direct dependency on `@wordpress/rich-text-control` from the fields package. It also standardizes how `placeholder` values are passed, reading them from `field.placeholder` instead of `config.placeholder`.

## Impact

- **Plugin & theme developers using DataViews:** Can now specify `Edit: 'richtext'` in field configurations to render rich-text inputs, eliminating the need to manually wire up `RichTextControl`.
- **`@wordpress/fields` consumers:** The `rich-text` and `title` fields now rely on the DataViews control; if you were relying on the previous per-field `edit.tsx` wrapper or direct `@wordpress/rich-text-control` imports in your own field packages, you'll need to adjust to the new control registration pattern.
- **Behavioral note:** The `placeholder` prop for rich-text fields is now sourced from `field.placeholder` rather than `config.placeholder`, aligning with other DataViews controls and fixing a previous inconsistency where the `title` field's placeholder was ignored.
- **No action required** for standard Gutenberg block or site editor usage, as the migration is internal to the packages.

## Technical details

- Adds `packages/dataviews/src/components/dataform-controls/richtext.tsx` and registers it under the `'richtext'` key in `FORM_CONTROLS`.
- Introduces `EditConfigRichText` type to handle rich-text-specific options like `allowedFormats` and `disableFormats`.
- `@wordpress/dataviews` now lists `@wordpress/rich-text-control` as a direct dependency, leveraging its existing transitive pull of `@wordpress/rich-text`.
- In `@wordpress/fields`, the `rich-text` and `title` field definitions switch from importing `RichTextControl` to using `Edit: 'richtext'`. The per-field `edit.tsx` wrapper is removed, and `@wordpress/fields` drops `@wordpress/rich-text-control` from its dependencies.
- The control reads `field.placeholder` directly, ensuring field-level placeholder values reach the underlying component.

## Contribution

Opened as a stacked PR on #78471, this change implements a reviewer suggestion to centralize the rich-text wrapper in DataViews rather than duplicating it across field packages. The approach was adopted to address dependency weight concerns raised during the initial PR review, and the PR was merged with co-authors @andrewserong and @ntsekouras. The discussion record contains no further design debates or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
