# #77942: Support `isAny` and `isNone` filter operators for numeric fields.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @widoz
- **Labels:** `[Type] Enhancement`, `[Package] DataViews`
- **Merged:** [`53d7861`](https://github.com/WordPress/gutenberg/commit/53d786161d780f82c849139f5851afde997a49c1)
- **Discussion:** [#77942](https://github.com/WordPress/gutenberg/pull/77942) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/dataviews` filter engine now honors the `isAny` and `isNone` operators when the field value is a scalar number. Previously, fields of type `integer`/`number` configured with these operators returned an empty result set because the operator matchers only handled arrays and strings. The fix extends the scalar branch to include `number`.

## Impact

**Plugin & theme developers using `@wordpress/dataviews`**
- Fields of type `integer` or `number` (e.g. an `author` ID field with `elements`) that declare `filterBy.operators: [ 'isAny', 'isNone' ]` now filter correctly through `filterSortAndPaginate`. Before, they matched nothing.
- No API changes, no deprecations, and no configuration or migration needed. Existing string and array behavior is unchanged.
- `isAll` is not touched by this change. The PR author questions whether `isAll` makes sense for scalar values, and the diff leaves it as is.

**Site owners / hosting / REST consumers**
- No action required. This affects client-side filtering in the DataViews package only.

## Technical details

The change is in `packages/dataviews/src/utils/operators.tsx`, in two places: the `isNoneOperatorDefinition.isMatch` function and the `isAny` entry in `OPERATORS`.

Before, each matcher had an array branch, then `else if ( typeof fieldValue === 'string' )` for the scalar case, and everything else fell through to `false`. Now the scalar branch also accepts `number`:

```js
// before
} else if ( typeof fieldValue === 'string' ) {
	return filterValue.includes( fieldValue );
}

// after
}

const fieldValueType = typeof fieldValue;
if ( fieldValueType === 'string' || fieldValueType === 'number' ) {
	return filterValue.includes( fieldValue );
}
```

`isNone` is the negation (`! filterValue.includes( fieldValue )`). The comparison uses `Array.prototype.includes`, so it is strict-equality: a numeric field value matches only numeric entries in `filterValue`, not numeric strings.

Tests were added in `packages/dataviews/src/utils/test/filter-sort-and-paginate.js`: `isAny` on `satellites` with `[16, 2]` returns 3 items, and `isNone` with `[0, 16, 2]` returns 7. A bug-fix entry was added to `packages/dataviews/CHANGELOG.md`.

## Contribution

@widoz opened the PR after finding that `filterSortAndPaginate` returned an empty collection for an integer `author` field with `isAny`/`isNone`/`isAll`, and he raised doubt about whether `isAll` belongs on scalar fields. @youknowriad reviewed it as looking good and pinged @ntsekouras and @oandregal. @oandregal later noted that reviewing operators led him to a follow-up, PR #82463.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
