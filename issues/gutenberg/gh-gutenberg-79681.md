# #79681: Icons: Filter the icon library picker by collection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `Needs Design Feedback`, `[Package] Block library`, `[Block] Icon`
- **Merged:** [`840eee7`](https://github.com/WordPress/gutenberg/commit/840eee758112779929319d4270319648ca66ad59)
- **Discussion:** [#79681](https://github.com/WordPress/gutenberg/pull/79681) · 12 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Icon block picker now filters icons by registered collection using a vertical tabs interface. This change improves usability for sites with large icon sets by scoping search and rendering to individual collections, and introduces asynchronous batch rendering to prevent UI freezing during load.

## Impact

- **Block & theme developers:** No breaking changes to public APIs or block.json schemas. However, internal component signatures changed: `CustomInserterModal` now accepts `onClose`, `value`, and `onChange` instead of `icons`, `setInserterOpen`, `attributes`, and `setAttributes`. `IconGrid` now uses `value` instead of `attributes`. Developers who have forked or heavily customized the icon inserter modal will need to update their component props.
- **Site owners & editors:** A faster, more organized icon picker with collection tabs and an "All" view. No configuration required.
- **Hosting & platform teams:** No action required. The change is contained within the block editor UI.

## Technical details

The diff refactors `packages/block-library/src/icon/components/custom-inserter/index.js` and `icon-grid.js` to introduce collection-based filtering and async rendering:
- **Data fetching:** Collections are resolved via `coreDataStore.getEntityRecords( 'root', 'iconCollection' )`. Icons are fetched per collection using `getEntityRecords( 'root', 'icon', { namespace: collectionSlug } )`, where `collectionSlug` defaults to the currently selected icon's collection or the first available collection.
- **Async rendering:** `useAsyncList` from `@wordpress/compose` is used with `BATCH_SIZE = 20` to render icons incrementally, preventing main-thread blocking.
- **UI components:** The modal now uses `Tabs.Root`, `Tabs.List`, `Tabs.Tab`, and `Tabs.Panel` from `@wordpress/ui` to display a vertical sidebar of collections alongside an "All" tab. Search input is scoped to the active tab.
- **Auto-scroll:** A `useLayoutEffect` hook checks if the selected icon is ready to scroll and calls `node.scrollIntoView( { block: 'center' } )` only if the container hasn't been manually scrolled, using `getScrollContainer` from `@wordpress/dom`.
- **Prop changes:** `CustomInserterModal` now receives `onClose`, `value`, and `onChange`. `IconGrid` receives `value` instead of `attributes` to determine the selected icon.
- **Styling:** `editor.scss` adds layout styles for `.wp-block-icon__inserter-sidebar`, `.wp-block-icon__inserter-panel`, and `.wp-block-icon__inserter-loading`, plus explicit SVG sizing rules to fit icons within a 24×24 grid.

## Contribution

Opened by @t-hamano and merged in commit `840eee7`, this PR is a follow-up to #72215 and part of #75715. Co-authors include @tyxla, @mirka, @Mamaduka, @ciampo, and @mahdialikhanusiya. Review feedback prompted the addition of an "All" tab and adjustments to the auto-scroll behavior to avoid interfering with user navigation. The bundling of `@wordpress/ui`'s `Tabs` component caused a noticeable bundle size increase, which reviewers noted should improve as more components share the base-ui dependency.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
