# #80570: Widget Dashboard: preserve widget chrome flex layout

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dognose24
- **Labels:** `[Type] Enhancement`, `First-time Contributor`, `[Package] Widget Dashboard`
- **Merged:** [`d36ca77`](https://github.com/WordPress/gutenberg/commit/d36ca772d5a84341828e911f083faa73bcf3773c)
- **Discussion:** [#80570](https://github.com/WordPress/gutenberg/pull/80570) · 6 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Widget Dashboard now explicitly sets `display: flex; flex-direction: column` on its chrome components via local module CSS. This prevents host environments that inject unlayered HTML5 element resets (e.g., `section { display: block; }`) from breaking the widget tile layout, which previously caused content clipping and overflow.

## Impact

- **Site owners & hosting platforms**: No action required. Fixes visual layout clipping in the Widget Dashboard when host-level CSS resets semantic elements.
- **Plugin & theme developers**: No code changes or migration steps needed. The change is internal to the `@wordpress/widget-dashboard` package and does not alter public APIs, component props, or block.json schemas.

## Technical details

The diff modifies two CSS module files in `packages/widget-dashboard/src/components/` to harden the flex-column layout against external resets:
- `widget-chrome.module.css`: Adds `.widget-chrome { display: flex; flex-direction: column; }`
- `widget-preview-chrome.module.css`: Adds `.card { display: flex; flex-direction: column; }`

The shared `Card` component applies its layout inside `@layer wp-ui.components`. Unlayered host resets override layered rules regardless of specificity, breaking the flex parent. By restating the flex layout on the module's own unlayered class, normal CSS specificity ensures the rule wins over element selectors like `section`. The content area already relies on this invariant (`flex: 1` + `min-height: 0`), so the change is a visual no-op in standard environments.

## Contribution

Opened by @dognose24 and reviewed by @retrofox. Initial feedback requested clearer testing instructions and questioned the scope. The author clarified that the issue stems from hosting platforms injecting legacy, unlayered HTML5 resets into `wp-admin`, provided a DevTools injection test plan, and confirmed the fix works via local specificity. @retrofox requested the same hardening be applied to the preview chrome, which the author implemented before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
