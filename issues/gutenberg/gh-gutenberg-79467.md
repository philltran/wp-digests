# #79467: Media Fields: Ensure the current post is always included in the initial options

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`ac52d25`](https://github.com/WordPress/gutenberg/commit/ac52d25129e2763b47275804f5a3cdad73f44c0a)
- **Discussion:** [#79467](https://github.com/WordPress/gutenberg/pull/79467) · 2 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

This pull request updates the Media Fields block’s “Attached to” control to always include the current post in the initial dropdown options. Previously, the initial suggestion list only reflected search results, which could exclude the post being edited if it did not match the default query. The change aligns this control’s behavior with similar editor controls like Post Author and Page Parent, improving consistency and reducing friction when attaching media.

## Impact

- **Plugin & theme developers**: No code changes required. The behavior change is internal to the Media Fields block.
- **Site owners & editors**: The “Attached to” dropdown in the media details modal now reliably shows the current post in the initial list, matching the behavior of other entity selectors.
- **No action required** for existing sites or custom blocks using this control.

## Technical details

The change modifies `packages/media-fields/src/attached_to/edit.tsx`. When fetching results for the attached-to selector, the component now maps the results to `suggestions` and checks whether the current `value` (the currently attached post ID) is absent from those results and whether no search filter is active (`!filterValue`). If both conditions are true, it appends `defaultPost` to the options array before calling `setOptions`. This ensures the current post is prepended to the suggestion list only on the initial load, preventing it from appearing in filtered search results.

```tsx
// Before
setOptions( mappedSuggestions );

// After
const includeCurrent = !filterValue && suggestions.findIndex( ( s ) => s.value === value ) === -1;
setOptions( suggestions.concat( includeCurrent ? defaultPost : [] ) );
```

## Contribution

Opened by @Mamaduka as a follow-up to #74432, with contributions from @andrewserong and @ramonjd. The PR implements a straightforward state-logic adjustment to align the “Attached to” control with existing editor patterns. No alternative approaches or significant design debates were recorded in the PR thread; the change was reviewed and merged directly.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
