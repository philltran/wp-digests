# #78470: Tooltip migration: dataviews consumers (3/5)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] DataViews`
- **Merged:** [`4251fca`](https://github.com/WordPress/gutenberg/commit/4251fcaf75002797a91c12c13d7155476f97e914)
- **Discussion:** [#78470](https://github.com/WordPress/gutenberg/pull/78470) · 4 comments · 0 reactions

## Summary

PR #78470 migrates four `Tooltip` call sites within `@wordpress/dataviews` from the legacy `Tooltip` exported by `@wordpress/components` to the new compositional `Tooltip` from `@wordpress/ui` (backed by base-ui). This is the third of five PRs in a coordinated migration series; parts 1–2 covering `@wordpress/block-editor`, `@wordpress/block-directory`, `@wordpress/editor`, `@wordpress/edit-post`, and `@wordpress/edit-site` had already landed. Three accessibility regressions that the legacy component papered over via `aria-describedby` injection are fixed inline as part of the migration.

## Impact

**Plugin & theme developers**
- No public API change to `@wordpress/dataviews` exports. This is an internal implementation refactor.
- Code that imports `Tooltip` directly from `@wordpress/ui` must carry `// eslint-disable-next-line @wordpress/use-recommended-components` until the migration series completes and the lint rule allow-list is updated in one go.

**Accessibility / end users**
- Three a11y fixes ship with this PR:
  - The reset/remove `×` button on an active filter chip now has an explicit `aria-label` matching its tooltip text, restoring an accessible name that was silently provided by the legacy `aria-describedby` mechanism.
  - Validation error messages in the DataForm panel label (`get-label-content`) are now exposed inline to assistive technology via a `<VisuallyHidden>` child.
  - The validation-error icon in `summary-button` now carries `role="img" aria-label={ errorMessage }` instead of relying on the tooltip for its accessible name.

**No action required** for sites consuming DataViews through the post editor, site editor, or dashboard — the shell-level `<Tooltip.Provider>` instances mounted in PRs 2 and 5 of this series cover the DataViews package automatically.

## Technical details

**Migrated files** (all under `packages/dataviews/src/`):
- `components/dataviews-filters/filter` — filter chip trigger (`<div role="button">`) and adjacent reset/remove `<button>`
- `components/dataform-layouts/panel/utils/get-label-content` — validation error tooltip on the field label `<span>`
- `components/dataform-layouts/panel/summary-button` — validation error tooltip on an icon `<span>`
- `components/dataviews-layouts/grid/composite-grid` — field-label tooltip on a `<FlexItem>`

**Import change** in each file:
```js
// Before
import { Tooltip } from '@wordpress/components';

// After
// eslint-disable-next-line @wordpress/use-recommended-components
import { Tooltip } from '@wordpress/ui';
```

**API change** — the legacy `Tooltip` wraps a single trigger child and accepts a `text` prop; the new compositional API from `@wordpress/ui` uses sub-components (`Tooltip.Trigger`, `Tooltip.Positioner`, `Tooltip.Content`). Because `placement="top"` is the default in the new API, the codemod emits no explicit `<Tooltip.Positioner>` for these four sites — visual placement is unchanged.

**Migration tooling** — the jscodeshift codemod landed in PR #78411 (`tools/codemods/tooltip-components-to-ui.js`) was run against the package:
```sh
npx jscodeshift -t tools/codemods/tooltip-components-to-ui.js \
    --extensions=js,jsx,ts,tsx --parser=tsx \
    packages/dataviews
```
Import placement was finalized by hand after the codemod pass.

**`<Tooltip.Provider>` not added** — `@wordpress/dataviews` is a library composed inside an editor shell; it relies on the shell-level providers mounted by PRs 2 and 5 of the series.

**Remaining scope** — `Tooltip` usages inside `@wordpress/components` itself (notably the internal `Tooltip` wrapping `Button` and `TooltipInternalContext`) are explicitly out of scope and tracked as a separate follow-up.

## Contribution

Opened and authored by @ciampo as part 3 of a 5-PR migration plan seeded by #78095 (new `@wordpress/ui` `Tooltip` API). Parts 1 (#78411) and 2 (#78466) had merged to trunk before this PR was opened. The PR carried 4 review comments (screenshot-per-site review approach noted in the description) and no substantive design debate — the migration pattern and eslint-disable strategy were already established in the earlier PRs. The author notes Cursor (Claude) assisted with authoring; all changes were human-reviewed and exercised locally before push.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
