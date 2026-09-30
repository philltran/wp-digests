# #81396: Add category filtering to the start page options patterns modal

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Block editor`, `[Package] E2E Tests`, `[Feature] Patterns`
- **Merged:** [`385aff5`](https://github.com/WordPress/gutenberg/commit/385aff52a7dc1fd923f5e5bcb122436f3668b77d)
- **Discussion:** [#81396](https://github.com/WordPress/gutenberg/pull/81396) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The "Choose a pattern" modal shown when creating a new page or post now includes a category sidebar and a search field, replacing the previous flat list. Only categories that contain at least one start pattern for the current post type are displayed, with labels pulled from registered pattern categories. The category-population logic was extracted from `usePatternCategories` into a shared `getPopulatedCategories` utility, and both it and the existing `searchItems` helper are exposed through the block-editor package's private APIs. When no start pattern belongs to a registered category, the modal falls back to the original flat list.

## Impact

- **Site owners / content editors:** The pattern selection modal gains a category sidebar and search box when patterns are registered with categories. No configuration change is needed; existing `register_block_pattern` and `register_block_pattern_category` calls are picked up automatically.
- **Plugin & theme developers:** No new public or experimental APIs are added. `getPopulatedCategories` and `searchItems` are exposed only through `privateApis` (unlocked via `unlock()`), so they are not a stable contract. The pattern explorer in the inserter is unchanged. No code changes are required.
- **No action required** for existing themes or plugins. Patterns restricted to other post types (e.g. `postTypes: ['post']`) and their categories are correctly excluded from the page modal and vice versa.

## Technical details

**New utility — `getPopulatedCategories`** (`packages/block-editor/src/components/inserter/block-patterns-tab/utils.js`):

```js
export function getPopulatedCategories( patterns, allCategories ) {
  const categories = allCategories
    .filter( ( category ) =>
      patterns.some( ( pattern ) =>
        pattern.categories?.includes( category.name )
      )
    )
    .sort( ( a, b ) => a.label.localeCompare( b.label ) );

  if (
    patterns.some( ( pattern ) => ! hasRegisteredCategory( pattern, allCategories ) ) &&
    ! categories.find( ( category ) => category.name === 'uncategorized' )
  ) {
    categories.push( { name: 'uncategorized', label: __( 'Uncategorized' ) } );
  }

  return categories;
}
```

The previously inline logic in `usePatternCategories` (including the local `hasRegisteredCategory` helper) was moved into this function; the hook now calls `getPopulatedCategories` directly.

**Private API exposure** (`packages/block-editor/src/private-apis.js`): `getPopulatedCategories` and `searchItems` (from `./components/inserter/search-items`) are added to the `lock()` call, making them available via `unlock( blockEditorPrivateApis )`.

**Modal component** (`packages/editor/src/components/start-page-options/index.jsx`):
- New `useStartPatternCategories` hook merges `__experimentalBlockPatternCategories` from block-editor settings with `getUserPatternCategories()` from core-data, then calls `getPopulatedCategories`. If every resulting category is `uncategorized`, it returns an empty array (no sidebar rendered).
- The modal renders `Tabs.Root` (orientation `vertical`) from `@wordpress/ui` with a `SearchControl` and `Tabs.List` in a sidebar `Stack`, and one `Tabs.Panel` per category containing `PatternSelection` or a "No results found." message.
- Filtering: category filter checks `pattern.categories?.includes( activeCategory )` (with special handling for `uncategorized`); search uses `searchItems( patterns, searchValue )`.
- A guard resets `activeCategory` to `allPatterns` if the selected category no longer exists (e.g. post type changed while modal is open).

**Responsive layout** (`packages/editor/src/components/start-page-options/style.scss`):
- The sidebar is hidden below `break-medium` and absolutely positioned at `$sidebar-width` above it.
- The pattern grid switches from viewport-based `column-count` breakpoints to a container query: `@container (min-width: #{2 * $pattern-min-width + $grid-unit-30})` with `columns: 4 $pattern-min-width` (where `$pattern-min-width: 220px`). The same container-query approach is applied to `start-template-options/style.scss`.

**E2E fixture** (`packages/e2e-tests/plugins/starter-page-patterns.php`): registers three pattern categories (`test-about`, `test-services`, `test-post-only`) and four patterns — two page-only, one uncategorized page, one post-only — to exercise filtering, search, and post-type scoping.

## Contribution

Opened by @jorgefilipecosta to close issue #56944. During review, @t-hamano flagged that PR #81807 had refactored the pattern modal sidebar to use `ui/Tabs`, creating a merge conflict; the conflict was resolved in commit `890db24f2ac`. The PR also received input from @glendaviesnz and @richtabor. The author noted that AI assistance was used in developing the PR. CodeRabbit's automated review flagged a docstring-coverage warning (12.5% vs. 80% threshold) on the new utility functions, which was not addressed before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
