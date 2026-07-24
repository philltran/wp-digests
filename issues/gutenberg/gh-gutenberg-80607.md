# #80607: Global Styles: Resolve link element styles in block inspector controls for blocks that are links

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Global Styles`, `[Feature] Design Tools`, `Backported to WP Core`
- **Merged:** [`d45ca55`](https://github.com/WordPress/gutenberg/commit/d45ca55268b8bbcf75b1032a74cc649f27b040e8)
- **Discussion:** [#80607](https://github.com/WordPress/gutenberg/pull/80607) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request updates the block editor’s Global Styles inheritance logic so that inspector controls for whole-block link blocks now reflect inherited `styles.elements.link` values from the site’s global styles. Previously, blocks that render entirely as `<a>` elements displayed empty or default typography and color controls in the inspector, even when link styles were configured globally. The fix folds the `link` element layer into the top-level controls for these specific blocks, aligning their editor experience with how `button` and heading blocks already handle their respective element styles.

## Impact

- **Block & theme developers:** No code changes required. The modification is internal to the block editor’s `inherited-value-context` provider and does not expose new public APIs or hooks.
- **Site owners & editors:** Inspector controls for link-based blocks (e.g., Read More, Login/out, pagination, and comment links) will now accurately display inherited link typography and color values set in Global Styles, complete with reset affordances when local overrides are applied.
- **No breaking changes, deprecations, or migration steps.** Existing block markup, `theme.json` schemas, and REST API responses remain unaffected.

## Technical details

The change modifies `packages/block-editor/src/components/global-styles/inherited-value-context.js`, specifically the `getElementLayers` function. It adds a `case` statement that returns `['link']` for a defined list of whole-block link blocks: `core/read-more`, `core/loginout`, `core/post-navigation-link`, `core/query-pagination-next`, `core/query-pagination-previous`, `core/query-pagination-numbers`, `core/comments-pagination-next`, `core/comments-pagination-previous`, `core/comments-pagination-numbers`, `core/comment-edit-link`, `core/comment-reply-link`, and `core/post-comments-link`.

Before this change, these blocks fell through to the `default` case and returned an empty array, causing the `useResolvedStyle` hook to skip folding `styles.elements.link` into their top-level `color.text` and `typography` controls:

```js
// Before
default:
    return [];
```

After the change, the function explicitly returns the `link` layer for the affected blocks:

```js
// After
case 'core/read-more':
case 'core/loginout':
// ... other link blocks ...
    return [ 'link' ];
default:
    return [];
```

The diff also adds unit tests in `packages/block-editor/src/components/global-styles/test/inherited-value-context.js` to verify that whole-block links correctly surface inherited link styles, while container blocks like `core/paragraph` correctly exclude the `link` layer to prevent color bleeding into their own text controls.

## Contribution

Opened by @andrewserong as a direct follow-up to earlier PRs that implemented the same inheritance resolution for heading elements (#77894, #80495). During review, @ramonjd and the author agreed to scope the implementation strictly to blocks that render entirely as `<a>` elements, explicitly deferring hybrid blocks (like `core/heading` or `core/site-title`) and server-side rendered link containers (like `tag-cloud` or `categories`) to future iterations. The PR was reviewed, merged, and cherry-picked to the `wp/7.1` branch for inclusion in the upcoming release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
