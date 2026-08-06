# #79329: DataViews: Make grid and table item preview aspect ratio configurable

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @vianasw
- **Labels:** `[Type] Enhancement`, `First-time Contributor`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`acf17cd`](https://github.com/WordPress/gutenberg/commit/acf17cde10d01dd99aadacfadfe3bc25be83e4b9)
- **Discussion:** [#79329](https://github.com/WordPress/gutenberg/pull/79329) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The DataViews package now accepts an `aspectRatio` configuration option for both `grid` and `table` layouts, replacing the previous hard-coded square preview with a consumer-set uniform ratio. The option defaults to `'1/1'` to preserve existing behavior, but supports presets like `'16/9'` and `'4/3'` to better match video thumbnails, wide templates, or other media types without breaking row alignment.

## Impact

- **Plugin & theme developers:** Can now pass `layout: { aspectRatio: '16/9' }` to `ViewGrid` or `ViewTable` configs to control preview dimensions. No immediate migration needed; existing views remain square.
- **Hosting & platform / Core admin:** The PHP REST schema for view configuration (`class-gutenberg-rest-view-config-controller-7-1.php`) does not yet include the new property, so server-side validation or UI exposure will require a follow-up PR.
- **No action required** for consumers who do not configure the option or rely on the default square previews.

## Technical details

Adds `aspectRatio?: MediaAspectRatio` to the `layout` shape in `ViewGrid` and `ViewTable` types (`packages/dataviews/src/types/dataviews.ts`). The value is validated against the `MEDIA_ASPECT_RATIOS` constant (`'1/1' | '4/3' | '3/4' | '3/2' | '2/3' | '16/9' | '9/16'`) defined in `packages/dataviews/src/constants.ts`.

- **Grid:** `composite-grid.tsx` injects `--wp-dataviews-media-aspect-ratio` as a CSS custom property on the grid container. The `.dataviews-view-grid__media` rule in `style.scss` reads it via `aspect-ratio: var(--wp-dataviews-media-aspect-ratio, 1/1)`.
- **Table:** `index.tsx` applies the same custom property to the table root and toggles a `has-media-aspect-ratio` modifier. Under that modifier, `style.scss` sets `height: var(--wpds-dimension-size-md)` and `aspect-ratio` on `.dataviews-column-primary__media`, while `column-primary.tsx` dynamically calculates a `sizes` hint for the media field to optimize srcset selection.

Existing consumers are unaffected because the custom property always defaults to `1/1`, and the table modifier only activates when `aspectRatio` is explicitly set.

**Usage pattern change:**
```diff
- const view = { type: 'grid', layout: { previewSize: 200 } };
+ const view = { type: 'grid', layout: { previewSize: 200, aspectRatio: '16/9' } };
```

## Contribution

Opened by first-time contributor `@vianasw` to address a long-standing request for configurable preview shapes. Reviewers `@ntsekouras`, `@jasmussen`, and `@oandregal` approved the uniform-ratio approach, deliberately avoiding a stalled user-facing toggle that risked visual rivers. `@ntsekouras` deferred PHP schema updates to a follow-up to avoid scope creep, and `@oandregal` noted a concurrent schema-generation PR that may require rebasing. The PR merged with co-author credits to `ntsekouras`, `jasmussen`, and `oandregal`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
