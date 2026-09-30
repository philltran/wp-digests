# #81352: Widget Dashboard: host-tunable tile spacing via public custom properties

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`, `[Package] Widget Dashboard`
- **Merged:** [`1a1b94c`](https://github.com/WordPress/gutenberg/commit/1a1b94c8a2b81e22bcb4101530a4a7da82882c0a)
- **Discussion:** [#81352](https://github.com/WordPress/gutenberg/pull/81352) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/widget-dashboard` package now exposes two public CSS custom properties, `--wp-widget-dashboard-tile-padding` and `--wp-widget-dashboard-tile-header-gap`, so a host can tune tile density. Card declares its spacing properties on `Card.Root` itself, so values set on an ancestor never reached the tile. The dashboard chrome now resolves the public properties into Card's `--wp-ui-card-padding` and `--wp-ui-card-header-content-gap` on the tile element. Defaults are unchanged.

## Impact

- **Plugin/host developers embedding the Widget Dashboard:**
  - You can set `--wp-widget-dashboard-tile-padding` (e.g. `var(--wpds-dimension-padding-lg)`) to get denser tiles without selectors against hashed class names.
  - `--wp-widget-dashboard-tile-header-gap` follows the padding unless set separately.
  - Use `--wpds-*` spacing tokens as values.
- **Where to set them:** The README says to set them at `:root`, not on a dashboard wrapper. The widget picker mounts in a dialog under `document.body`, which a wrapper's custom properties never reach, so previews would not follow the override.
- **Everyone else:** No action required. With no override, tiles render exactly as before, and there are no breaking changes or deprecations.

## Technical details

The change is CSS-only, in three module stylesheets, plus README and CHANGELOG entries.

- `widget-chrome.module.css`: `.widget-chrome` now declares:
  ```css
  --wp-ui-card-padding: var(--wp-widget-dashboard-tile-padding, var(--wpds-dimension-padding-2xl));
  --wp-ui-card-header-content-gap: var(--wp-widget-dashboard-tile-header-gap, var(--wp-widget-dashboard-tile-padding, var(--wpds-dimension-gap-xl)));
  ```
- `widget-header.module.css`: the `.overlay` (floating header for full-bleed tiles) changes its hardcoded `--wp-ui-card-padding: var(--wpds-dimension-padding-2xl)` to use `var(--wp-widget-dashboard-tile-padding, …)`, keeping the toolbar aligned with in-card headers.
- `widget-preview-chrome.module.css`: the preview `Card.Root` element gets the same two declarations, so picker previews follow host density.

The fallback chain is deliberately repeated at each site rather than resolved once on a shared ancestor, since that would recreate the inheritance problem. Module CSS is unlayered, so it wins over Card's `@layer wp-ui` declarations. The header-gap fallback chain falls back to the tile padding, which avoids a double space between header and body when only padding is lowered.

Example host override:
```css
:root {
	--wp-widget-dashboard-tile-padding: var(--wpds-dimension-padding-lg);
}
```

## Contribution

A reviewer (Chi, per the reply) asked about the default density. @retrofox kept the Card default on purpose. System-level density is still an open upstream question (#74556), an earlier implementation was dropped (#78741), and the range being explored is 16–20px rather than a fixed 16px. The properties act as a bridge so a host can run that experiment, and they would be remapped if a Card density prop lands later. Follow-ups listed in the PR: decide the dashboard's default density with design, and adopt the outcome of the Card spacing contract issue (#81351).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
