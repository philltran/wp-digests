# #81378: DataViews:  Fix `array` field type validation for empty values

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`0f7b5c4`](https://github.com/WordPress/gutenberg/commit/0f7b5c4eef49315a73ed643a54ec38c7e6d73ee5)
- **Discussion:** [#81378](https://github.com/WordPress/gutenberg/pull/81378) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The built-in validation for the DataViews `array` field type threw on empty values (`undefined`, `''`, `null`), and that exception surfaced as a validation error. As a result, a form with an empty optional array field was always invalid. The fix makes `isValidCustom` return `null` (valid) for empty values, consistent with other field types. Non-empty enforcement is now left to the `required` rule.

## Impact

**Plugin & theme developers using DataForm / DataViews**
- Forms with an empty, optional `type: 'array'` field no longer report as invalid, so `Submit` buttons gated on `isValid` are enabled on load.
- If you relied on the old (accidental) failure to block empty arrays, add the `required` rule instead.
- Fields that declare `elements` are **not** fixed by this change. The default-on `elements` rule still rejects empty values (`isValidElements` does `[].concat( value )`, producing `[ undefined ]`). Only array fields without `elements`, or with `isValid.elements: false`, benefit. A follow-up is planned.

**Site owners / hosting / REST consumers:** No action required.

## Technical details

The change is in `packages/dataviews/src/field-types/array.tsx`, in `isValidCustom`. Previously the check was:

```ts
if (
	! [ undefined, '', null ].includes( value ) &&
	! Array.isArray( value )
) {
	return __( 'Value must be an array.' );
}
```

Now empty values return early, and the array check is separate:

```ts
// Allow empty values; use the `required` rule to enforce non-empty ones.
if ( [ undefined, '', null ].includes( value ) ) {
	return null;
}

if ( ! Array.isArray( value ) ) {
	return __( 'Value must be an array.' );
}
```

The diff shows only this early return changing in the function. The remainder of `isValidCustom` (including the per-item string check, which the new tests exercise with the message `Every value must be a string.`) is outside the diff context and unchanged.

Tests were added to `packages/dataviews/src/hooks/test/use-form-validity.ts`:
- An empty (`undefined`) array value yields `validity` of `undefined` and `isValid === true`.
- `[ 'valid', 42 ]` yields a `custom` invalid result with `Every value must be a string.`

A `packages/dataviews/CHANGELOG.md` entry was also added. No new hooks, schema, or DB changes.

## Contribution

Extracted by @ntsekouras from the larger PR #81377. Review feedback (credited to @jorgefilipecosta in the props list) noted that fields declaring `elements` still fail on empty values because `isValidElements` could also treat empty values as valid. The author agreed this affects all field types with `elements` and deferred it to a follow-up rather than widening this PR. The changelog was reworded to note that limitation before merge. The PR description states it was generated with an AI tool and manually reviewed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
