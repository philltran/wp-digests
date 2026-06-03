# #78231: Visually align `Notice` from `@wordpress/components` with `Notice` from `@wordpress/ui`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Notices`, `[Package] Block editor`, `[Package] Edit Post`, `Design System`
- **Merged:** [`7a944ff`](https://github.com/WordPress/gutenberg/commit/7a944ff3b03a9785010eeaf6af10a701a64a49ed)
- **Discussion:** [#78231](https://github.com/WordPress/gutenberg/pull/78231) · 10 comments · 1 reactions

## Summary

The `Notice` component in `@wordpress/components` has been visually refreshed to match the `Notice` in `@wordpress/ui`: the legacy 4px left-stripe is replaced with a 1px border, rounded corners, and intent-tinted card surfaces driven by WPDS design tokens. Layout switches to CSS grid, placing the dismiss button in its own column and hoisting `.components-notice__actions` out of `.components-notice__content` into a sibling row. `InlineNotices` in `@wordpress/notices` gains a `.components-inline-notices` stack wrapper that spaces pinned and dismissible notice lists, plus an optional `className` prop; legacy per-package Notice CSS overrides have been deleted across `@wordpress/block-editor`, `@wordpress/block-library`, `@wordpress/list-reusable-blocks`, and the media replace flow. Part of the design-system convergence tracked in [#76135](https://github.com/WordPress/gutenberg/issues/76135).

## Impact

**Plugin and theme developers with custom Notice CSS — action required:**
- **Breaking:** The `is-dismissible` class is no longer added to the root `.components-notice` element. Replace selectors that target it with `:has(.components-notice__dismiss)` or equivalent.
- **Breaking:** `.components-notice__actions` is now a *sibling* of `.components-notice__content`, not a descendant. Selectors like `.components-notice__content .components-notice__actions` will not match. Update to `.components-notice__actions` directly.
- CSS targeting the old left-stripe visual (e.g. `border-left`, flat `background-color` on `.components-notice`) will produce unexpected results; audit and remove those overrides.

**Consumers of `InlineNotices` (`@wordpress/notices`) — minor action may be required:**
- `InlineNotices` no longer ships its own Notice stylesheet; it now depends on `@wordpress/components` styles for Notice chrome. Ensure `@wordpress/components` styles are loaded in any custom context that uses `InlineNotices`.
- `InlineNotices` gains an optional `className` prop applied to the new `.components-inline-notices` wrapper.
- `InlineNotices` returns `null` when both lists are empty (previously may have rendered empty shell markup).

**`edit-post` / classic meta box integrations:**
- The meta box height recalculation now queries `:scope > .components-inline-notices` instead of bare `.components-notice-list` siblings. Custom integrations that replicate this measurement logic need the same update.

**Hosting & platform / dark mode:**
- Notice surfaces now respond to WPDS tokens (`--wpds-color-bg-surface-{intent}-weak`, `--wpds-color-stroke-surface-{intent}`, `--wpds-color-fg-content-{intent}`). Dark-mode theming should be verified if your environment customises these tokens.

**Site owners / end users:** Visual change only — bordered, rounded card treatment replaces the coloured left-stripe. No action required.

## Technical details

**Container** — `.components-notice` replaces its left-stripe (`border-left: 4px solid`) and flat background with `border: 1px solid`, `border-radius: var(--wpds-border-radius-lg)`, and a tinted background. Per-intent colors are resolved through locally-scoped custom properties (`--wp-components-notice-background`, `--wp-components-notice-border-color`, `--wp-components-notice-color`) set from WPDS semantic tokens (`--wpds-color-bg-surface-{intent}-weak`, `--wpds-color-stroke-surface-{intent}`, `--wpds-color-fg-content-{intent}`) where `{intent}` ∈ `info | success | warning | error`.

**Layout** — `.components-notice` is now `display: grid` with two columns (content / dismiss) and two rows (content+dismiss / actions):

```css
/* Conceptual new grid */
.components-notice {
  display: grid;
  grid-template-columns: 1fr auto;
}
.components-notice__content  { grid-area: 1 / 1; }
.components-notice__dismiss  { grid-area: 1 / 2; }
.components-notice__actions  { grid-area: 2 / 1; }
```

**Markup — structural breaking change:**

```html
<!-- Before: actions nested inside content -->
<div class="components-notice is-dismissible">
  <div class="components-notice__content">
    Message
    <div class="components-notice__actions">…</div>
  </div>
  <button class="components-notice__dismiss">…</button>
</div>

<!-- After: actions are a sibling of content; no is-dismissible class -->
<div class="components-notice">
  <div class="components-notice__content">Message</div>
  <button class="components-notice__dismiss">…</button>
  <div class="components-notice__actions">…</div>
</div>
```

**Dismiss button** — uses the `closeSmall` icon; hover/active colors via `--wpds-color-fg-interactive-neutral` instead of legacy button accent-on-hover styles.

**Action buttons** — rendered at `size="compact"` to align with `@wordpress/ui` Notice action sizing.

**Typography** — `font-family`, `font-size`, and `line-height` are now set on the root `.components-notice` element (previously only on `__content`), so both `__content` and the sibling `__actions` row inherit the same WPDS body type scale. This also ensures `1lh` in vertical-alignment calculations resolves correctly.

**Three-layer inline notice spacing (no changes to `Notice` itself):**

| Layer | Selector | What it does |
|---|---|---|
| List | `.components-notice-list` | `display: flex; flex-direction: column; gap: var(--wpds-dimension-gap-md)` between individual notices |
| Stack | `.components-inline-notices` | Same gap between the pinned list and the dismissible list |
| Surface | `.editor-notices` | `padding: var(--wpds-dimension-padding-md)` — inset from canvas edges |

**Deleted CSS overrides** — legacy `Notice` override stylesheets removed from: `@wordpress/notices` (`InlineNotices` inline stylesheet), `@wordpress/block-editor` (content/style sheets), `@wordpress/block-library` (gallery editor, block placeholder, media replace flow), `@wordpress/list-reusable-blocks` (import form).

**New build artifact** — `build/styles/notices/style.css` (and `-rtl` variant, ~121 B) ships the `.components-inline-notices` wrapper styles from `@wordpress/notices`. The `@wordpress/notices` JS bundle shrank by ~170 B (−9%) as the inline stylesheet was removed.

**`edit-post` meta box measurement** — the resize observer that accounts for inline notice height was updated from querying `.components-notice-list` sibling elements to `:scope > .components-inline-notices`.

`isDismissible`, `onRemove`, `actions`, `spokenMessage`, and all other `Notice` props are functionally unchanged. `useSpokenMessage` politeness behaviour is unchanged.

## Contribution

PR opened and authored by @jameskoster as part of the broader `@wordpress/ui` design-system convergence tracked in issue [#76135](https://github.com/WordPress/gutenberg/issues/76135). The PR bundled the visual refresh with follow-up cleanup of legacy override CSS across five packages, spacing work on `InlineNotices`, and the meta box measurement fix. It drew 10 comments, primarily around the three-layer spacing approach and the decision to put spacing on list/stack containers rather than on individual `Notice` surfaces. Merged at [`7a944ff`](https://github.com/WordPress/gutenberg/commit/7a944ff3b03a9785010eeaf6af10a701a64a49ed). The author notes the work was produced with Cursor.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
