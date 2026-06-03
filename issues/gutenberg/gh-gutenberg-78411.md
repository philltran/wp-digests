# #78411: Tooltip migration: block-editor + block-directory consumers (1/5)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`
- **Merged:** [`96fa2d3`](https://github.com/WordPress/gutenberg/commit/96fa2d37fb94203a167a0b827d5874c14cf0899c)
- **Discussion:** [#78411](https://github.com/WordPress/gutenberg/pull/78411) · 4 comments · 0 reactions

## Summary

PR #78411 migrates 7 `Tooltip` call sites across `@wordpress/block-directory` and `@wordpress/block-editor` from the legacy flat API in `@wordpress/components` to the new compositional `Tooltip` API in `@wordpress/ui` (backed by base-ui). This is part 1 of a 5-PR series following #78095, which introduced the new API. The PR also lands a jscodeshift codemod at `tools/codemods/tooltip-components-to-ui.js` to automate the migration across all five PRs. No visual change is intended — placement, text, and hover/focus behavior are preserved.

## Impact

**Plugin and theme developers**
- No immediate action required. The `Tooltip` export from `@wordpress/components` is **not deprecated** in this PR; `Button`'s internal tooltip and `TooltipInternalContext` remain on the legacy implementation.
- `aria-describedby` is **no longer injected** into the 7 migrated trigger elements. Those tooltips are now visual-only. If custom code was relying on `aria-describedby` wiring from `block-patterns-list` (`WithToolTip`), `inspector-controls-tabs`, `media-preview`, `shadow-panel-components`, `preset-input-control/custom-value-controls`, `downloadable-block-list-item`, or `block-select-button`, that relationship is gone.

**Block developers using `@wordpress/block-editor` / `@wordpress/block-directory` internals**
- The `WithToolTip` wrapper in `block-patterns-list` and the six other sites now render the `@wordpress/ui` sub-component tree instead of a single `<Tooltip>` element. Any snapshot tests or DOM-structure assertions against those components will need updating.

**Hosting and platform (bundle-size sensitivity)**
- `build/scripts/block-directory/index.min.js` grows +32.7 kB (+317%) and `build/scripts/block-editor/index.min.js` grows +30.3 kB (+8.7%) due to the `@wordpress/ui` / base-ui dependency being pulled in for the first time in these packages.

**Accessibility**
- The `visibilityLabel` tooltip in `block-editor/components/list-view/block-select-button` is attached to a `<span aria-hidden="true">` trigger — a pre-existing condition that already made the label unreachable for assistive technology. This PR preserves that behavior unchanged and adds a `TODO` comment flagging it for a dedicated a11y follow-up.

## Technical details

The migration replaces the flat single-element API with a three-part sub-component structure. The `placement` prop is decomposed into `side` and `align` on `<Tooltip.Positioner>`; `text` becomes the `Tooltip.Popup` children; `className` moves to `Tooltip.Popup`.

```tsx
// Before — @wordpress/components
import { Tooltip } from '@wordpress/components';
<Tooltip text="Save" placement="top-end" className="my-tip">
  <Button />
</Tooltip>

// After — @wordpress/ui
import { Tooltip } from '@wordpress/ui';
<Tooltip.Root>
  <Tooltip.Trigger render={ <Button /> } />
  <Tooltip.Popup
    positioner={ <Tooltip.Positioner side="top" align="end" /> }
    className="my-tip"
  >
    Save
  </Tooltip.Popup>
</Tooltip.Root>
```

`Tooltip.Trigger` renders a `<button>` by default; the `render` prop hands off ref/event wiring to the underlying element via base-ui — no wrapper node is added. `aria-describedby` is not replicated.

No `Tooltip.Provider` is emitted at these call sites. A single shell-level `<Tooltip.Provider>` — needed for group coordination (instant-open after first hover) — is deferred to PRs 2 and 5 of the series.

Each migrated import carries a per-site `// eslint-disable-next-line @wordpress/use-recommended-components` directive because `@wordpress/ui`'s `Tooltip` is not yet on the Storybook recommendation allowlist.

**Codemod** (`tools/codemods/tooltip-components-to-ui.js`):

```sh
npx jscodeshift -t tools/codemods/tooltip-components-to-ui.js \
    --extensions=js,jsx,ts,tsx --parser=tsx \
    packages/<scope>
```

The transform rewrites named and aliased `Tooltip` specifiers (e.g. `Tooltip as WCTooltip`) from `@wordpress/components` to `@wordpress/ui` and emits the full `Tooltip.Root / Tooltip.Trigger / Tooltip.Popup` tree. It bails on the **entire file** without modifying it — printing a warning — when any usage in that file contains `shortcut`, `delay`, `hideOnClick`, `position`, spread attributes, multiple children, dynamic `placement`, or any unrecognised attribute. Those sites are handled manually in later PRs.

**Migrated files:**
- `packages/block-directory/src/components/downloadable-block-list-item`
- `packages/block-editor/src/components/block-patterns-list` (the `WithToolTip` helper component)
- `packages/block-editor/src/components/global-styles/shadow-panel-components`
- `packages/block-editor/src/components/inserter/media-tab/media-preview`
- `packages/block-editor/src/components/inspector-controls-tabs`
- `packages/block-editor/src/components/list-view/block-select-button`
- `packages/block-editor/src/components/preset-input-control/custom-value-controls`

## Contribution

@ciampo authored and merged this PR (commit `96fa2d3`) as a direct follow-up to #78095. It is the first installment in a planned 5-PR series (block-editor + block-directory → editor + edit-post + edit-site → dataviews → fields + media → boot) to consolidate all Gutenberg editor tooltip usage under `@wordpress/ui`. The PR description notes it was authored with Cursor (Claude) assistance and reviewed by a human before merge. Discussion was light (4 comments, 0 reactions) with no recorded design debate in the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
