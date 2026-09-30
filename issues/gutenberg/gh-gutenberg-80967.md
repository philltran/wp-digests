# #80967: Post Taxonomies: Replace FormTokenField in the flat term selector

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `Needs Accessibility Feedback`
- **Merged:** [`2111d47`](https://github.com/WordPress/gutenberg/commit/2111d472dd06703dcf19b66817a04c1ffed6a70b)
- **Discussion:** [#80967](https://github.com/WordPress/gutenberg/pull/80967) · 31 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The editor's flat-term selector (`PostTaxonomiesFlatTermSelector`, used for tags and other non-hierarchical taxonomies) no longer renders `FormTokenField`. It now uses the new `SearchableChipSelectControl` from `@wordpress/ui`. Term creation is now an explicit "Create: <name>" option in the dropdown rather than a side effect of pressing Enter or typing a comma, and term-to-ID assignment no longer relies on name lookups. The PR describes itself as an experiment to battle-test the new component in the WP environment.

## Impact

**Site owners / editors**
- Tag entry looks and behaves differently: selected terms are chips with remove buttons, suggestions come from a REST search, and a new term is created by choosing the "Create: <name>" item (shown only when the typed name has no exact match and the user has the create capability).
- The comma shortcut for creating several terms at once is removed. The PR author cites accidental term creation that required manual cleanup as the reason.
- New terms appear immediately as pending chips and are assigned once the REST request resolves; failures show a snackbar error and remove the pending chip.

**Plugin & theme developers**
- No API removals are listed, but anything relying on the `FormTokenField` DOM, roles, or keyboard flow inside the taxonomy panel (custom CSS, E2E tests) may break. The core E2E specs were updated: the combobox is filled, then the `option` named `Create: <name>` is clicked instead of pressing Enter, and the remove button is now named `Remove <term name>` rather than `Remove Mode`.
- Custom taxonomies still get their labels (`labels.singular_name`, `labels.not_found`, add-new label) applied.
- `PostTaxonomiesFlatTermSelector` remains wrapped in `withFilters`, so the existing filter-based replacement mechanism still works.

**Other**
- `build/scripts/editor/index.min.js` grows by about 22.3 kB (+3.86%) per the size-check bot.
- Reviewers noted the component carries a "Use with caution" status, with open focus-style issues (#80405, #80417) and an upcoming breaking change to `creatableItem` handling (#80989).

## Technical details

The change is in `packages/editor/src/components/post-taxonomies/flat-term-selector.jsx`.

- **Item model:** terms are mapped with `termToItem` to `{ value: String(term.id), label: unescapeString(term.name) }`. `isSameTerm` compares by `value`. `termNamesToIds`, the name-based `isSameTermName` lookup, and `unescapeTerm` usage are removed.
- **Search:** the `getEntityRecords` `useSelect` is replaced with an imperative `registry.resolveSelect( coreStore ).getEntityRecords( 'taxonomy', slug, { ...DEFAULT_QUERY, search } )`, debounced by 500 ms. `lastSearchRef` discards stale responses. Suggestions are cleared on every input change, and `filter={ null }` disables client-side filtering. `isSearching` drives a `Spinner` "Searching…" status, and a visually hidden `_n()` result count is announced.
- **Creation:** a `creatableItem` with value `__create__` is appended to `items` when `hasCreateAction`, the trimmed input is non-empty, there is no case-insensitive exact match among suggestions or selected values, and no search is in flight. `createTerm( name )` adds an optimistic pending item (`__pending__:<n>` value, tracked in `pendingTermsRef`), then calls `findOrCreateTerm` (which still handles `term_exists` by using `error.data.term_id`). On failure it calls `createErrorNotice` and drops only the pending chip. If the chip was removed while the request was in flight, the result is not assigned. On success the pending item is swapped for the saved term, de-duplicated, and assigned.
- **Assignment:** new `assignTerm( termId )` reads the current edited attribute via `registry.select( editorStore ).getEditedPostAttribute( taxonomy.rest_base )` so parallel creations and `MostUsedTerms` selections don't overwrite each other. `onChange` filters out pending items before calling `onUpdateTerms`. `appendTerm` (most-used) now uses `assignTerm`.
- **Capabilities:** `hasCreateAction` and `hasAssignAction` are now booleans from `post._links`. The selector returns early (no term fetching) when the user lacks the assign action.
- **Announcements:** `speak()` announces added/removed labels from `onChange` based on selection length change, and after creation completes.
- **UI:**

```jsx
<SearchableChipSelectControl
	openOnInputClick={ false }
	filter={ null }
	items={ items }
	value={ values }
	onValueChange={ onChange }
	isItemEqualToValue={ isSameTerm }
	inputValue={ inputValue }
	onInputValueChange={ onInputValueChange }
	showClearButton={ false }
	chipsContent={ ... <SearchableChipSelectControl.ChipWithRemove /> ... }
/>
```

The `@wordpress/editor` CHANGELOG gains an Enhancements entry. E2E specs `custom-taxonomies.spec.js` and `taxonomies.spec.js` were updated (the latter with helpers such as `defer()`, `openTaxonomyPanel`, `getTagChip`); the diff for the latter is truncated in the input.

## Contribution

Authored by @Mamaduka, with a disclosure that it was assisted by Claude, and it closes #15406, #30931 and #73782. @mirka (Components) agreed it was a good place to battle-test the "Use with caution" component and listed remaining blockers, and @Mamaduka judged only the `creatableItem` change (#80989) potentially blocking. @jasmussen's design review raised that the caret sits below the chips, that only one tag stayed selected at a time (a porting bug the author acknowledged), and that the "Create" popover can grow wide with long names. He also asked whether the create popover is needed at all versus mimicking trunk's compact behavior, and wanted chips closer to existing ones in size. The author defended dropping comma-to-create as an intentional UX change tied to #15406. The PR carried a `Needs Accessibility Feedback` label.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
