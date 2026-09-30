# #83323: Editor: Use SearchableSelect for the post author field

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Editor`
- **Merged:** [`d1180ee`](https://github.com/WordPress/gutenberg/commit/d1180ee9bdaba571cf84447a2db8975c6b57ab9b)
- **Discussion:** [#83323](https://github.com/WordPress/gutenberg/pull/83323) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `PostAuthor` component in `@wordpress/editor` now renders `SearchableSelect` from `@wordpress/ui` unconditionally, replacing the previous logic that switched between `ComboboxControl` (≥ 25 users) and `SelectControl` (< 25 users). Author search is performed entirely server-side via the REST API (`filter={ null }`), eliminating the client-side filtering that could produce incorrect results. The `(No author)` option is removed in favor of the select's placeholder text.

## Impact

- **Site owners / editors:** The author field in the post sidebar is now always a searchable dropdown, even on sites with fewer than 25 users. The `(No author)` option no longer appears; an unassigned author shows the "Select author" placeholder instead.
- **Plugin & theme developers:** The public `PostAuthor` export from `@wordpress/editor` is unchanged in name and signature. However, the internal `useAuthorsQuery` hook now returns `{ items, value, isLoading }` instead of `{ authorId, authorOptions, postAuthor, isLoading }`. The internal files `combobox.jsx` and `select.jsx` are deleted. Any code reaching into those internals will break.
- **No action required** for standard usage of the `PostAuthor` component. No new configuration, migration, or code changes are needed.

## Technical details

The diff deletes `packages/editor/src/components/post-author/combobox.jsx` and `select.jsx`, and rewrites `index.jsx` to render a single `SearchableSelect` from `@wordpress/ui`.

Key behavioral changes in the new `index.jsx`:

- `filter={ null }` disables client-side filtering; all narrowing happens via the REST `getUsers` query.
- A 300 ms `useDebounce` (from `@wordpress/compose`) gates the `search` state that feeds `useAuthorsQuery`.
- While a request is in flight or the input lags the debounced search, `authors` is set to an empty array so stale results are not shown.
- `onValueChange` calls `editPost( { author: Number( author.value ) } )`.
- The `isItemEqualToValue` prop uses a `isSameAuthor` helper comparing `value` (a stringified ID).

The `useAuthorsQuery` hook (`hook.js`) is refactored:

```js
// Before
return { authorId, authorOptions, postAuthor, isLoading };

// After
return { items, value, isLoading };
```

A new `authorToItem` helper maps each author to `{ value: String( author.id ), label: decodeEntities( author.name ) }`. The current author is prepended to `items` only when `search` is falsy and the author is not already in the fetched list. The old `(No author)` entry (value `0`) is removed entirely.

In `panel.jsx`, the import changes from `PostAuthorForm` to `PostAuthorControl`, and the `onClose` prop is no longer forwarded to the control.

Bundle impact: +1.37 kB in `build/scripts/editor/index.min.js` (+0.22 %).

## Contribution

Opened by @Mamaduka closing #76986, with co-authorship from @jasmussen, @mirka, @dd32, @youknowriad, and @annezazu. The PR notes AI assistance (Claude). The record carries no substantive design debate or rejected alternatives beyond the stated rationale that conditional rendering between two control types no longer makes sense once search is fully REST-driven.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
