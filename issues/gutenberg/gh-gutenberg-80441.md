# #80441:  [7.1 Backport] --  ContentEditableControl: fix invalid label association with contenteditable div 

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Package] DataViews`
- **Merged:** [`0b340fe`](https://github.com/WordPress/gutenberg/commit/0b340fe7034a6f9b149d8c5a167398f9c8faae7c)
- **Discussion:** [#80441](https://github.com/WordPress/gutenberg/pull/80441) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `ContentEditableControl` component in `@wordpress/components` now associates its label using `aria-labelledby` instead of an invalid `label[for]` attribute on a non-form `div`. This resolves Chrome console errors triggered by the invalid HTML association and ensures proper accessibility semantics for screen readers.

## Impact

- **Plugin & theme developers using `@wordpress/components`:** If you render `ContentEditableControl` directly, the label will no longer use `label[for]`. Instead, it uses `aria-labelledby` and explicitly focuses the editable div on click. No code changes are required unless you were relying on the previous DOM structure or `label[for]` behavior.
- **DataViews users:** The `RichTextControl` in DataViews inherits this fix, so console errors in data forms will cease.
- **No action required** for most consumers; the change is a transparent accessibility and HTML validity fix.

## Technical details

In `packages/components/src/content-editable-control/index.tsx`, the component now manually renders the label using `<BaseControl.VisualLabel>` (or `<VisuallyHidden>` when `hideLabelFromVision` is true) with a generated `id` (`${ controlProps.id }__label`). The editable `div` receives `aria-labelledby={ labelId }` instead of `aria-label={ label }`, and the `label[for]` association is removed. To preserve the click-to-focus behavior previously handled by the browser, an `onClick` handler is added to the label that calls `editableRef.current?.focus()`. The component also merges the internal `editableRef` with the `forwardedRef` using `useMergeRefs` from `@wordpress/compose`. Tests confirm the label no longer has a `for` attribute, the textbox correctly resolves its accessible name via `aria-labelledby`, and clicking the label focuses the field.

## Contribution

This change is a backport of PR #80344 to the `wp/7.1` branch. The automated cherry-pick failed due to a `CHANGELOG.md` conflict, which was manually resolved by keeping the release branch's changelog structure and inserting only this fix. The implementation was authored by @adamsilverstein with co-authorship from @t-hamano, and it directly addresses the accessibility issue reported in #80204. The source diff is byte-identical to the trunk merge, with only the changelog adjusted for the release branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
