# #81377: DataForm: Fix hidden fields validation

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`126a840`](https://github.com/WordPress/gutenberg/commit/126a840c69eed7a89e3927f64dcceb40f43f396f)
- **Discussion:** [#81377](https://github.com/WordPress/gutenberg/pull/81377) · 2 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

`useFormValidity` in `@wordpress/dataviews` now skips validation for fields hidden through the `isVisible` API. Before, a hidden field with rules such as `isValid.required` could keep the whole `DataForm` invalid even though the user could not see or fill it. Validation is re-run when the field becomes visible again, and any in-flight async validation for a field that just got hidden is discarded.

## Impact

- **Plugin & theme developers using `DataForm`:** Forms with conditionally visible fields (`isVisible`) that also declare `isValid` rules will now report valid while those fields are hidden. Submit buttons gated on `isValid` from `useFormValidity` will enable as expected. No code changes needed. Workarounds that stripped validation rules or cleared values when hiding a field can likely be removed.
- **Behavior to be aware of:** Only leaf fields are affected. Combined fields (fields with `children`) are always rendered by `DataFormLayout`, so they are always validated. Hidden children inside a combined field are skipped, while visible siblings are still validated.
- **Site owners / hosting / REST consumers:** Not affected.
- No deprecations or API removals.

## Technical details

All changes are in `packages/dataviews/src/hooks/use-form-validity.ts`.

- **New helper `isFormFieldHidden( formField, item )`:** returns `true` only when `formField.children.length === 0`, the field defines `isVisible`, and `isVisible( item )` returns false. The comment notes `DataFormLayout` applies `isVisible` only to leaf fields.
- **`validateFormField`:** returns `undefined` early for hidden fields, before any `required`, `custom`, or elements checks. Before returning, it increments `customCounterRef.current[ formField.id ]` and `elementsCounterRef.current[ formField.id ]` when they are set, so pending async results become stale and cannot resurface for a field that is no longer rendered.
- **`getFormFieldValue`:** the snapshot now includes `isHidden`. Leaf fields return `{ value, isHidden }` instead of the raw value, and combined fields return `{ value, isHidden, children }`. The `validate` step compares snapshots with `fastDeepEqual`, so folding in visibility makes a hidden→visible toggle with an unchanged value count as a change and forces re-validation. The unreachable `if ( ! childrenValues )` branch is removed.

Tests were added in `hooks/test/use-form-validity.ts` covering hidden fields, toggling visibility with an unchanged value, hidden children of combined fields, and a stale async `custom` result after hiding. The `validation` Storybook story gained `showConditionalText` and `conditionalText` fields. The README now states that hidden fields are not validated, and the CHANGELOG has a Bug Fix entry.

The PR description also mentions an inconsistency between `array` type fields and custom validation. It says this will be extracted into a separate PR, and the provided diff contains no `array`-specific change.

## Contribution

The bug surfaced while working on the content types experiment (issue #77600), which has since been removed. @ntsekouras authored the fix, and the PR notes it was AI-assisted and manually reviewed. The bot credits @jorgefilipecosta as an interacting account. The visible discussion is only bot comments (props and bundle size, about +255 B), with no design debate recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
