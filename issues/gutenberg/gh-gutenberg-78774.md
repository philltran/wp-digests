# #78774: Icons: maintain absolute stroke-width regardless of icon-size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Icons`, `[Feature] Icons`
- **Merged:** [`86c5139`](https://github.com/WordPress/gutenberg/commit/86c513917e9e268a4ff8dec6538f9f15fcdc447f)
- **Discussion:** [#78774](https://github.com/WordPress/gutenberg/pull/78774) · 10 comments · 1 reactions

## Summary

This PR lays the groundwork for migrating the `@wordpress/icons` set from fill-based to stroke-based SVGs without breaking existing consumers. The actual code change is narrow: `square.svg` (the only stroke-based icon already in trunk) is updated to a new authoring convention where `stroke`, `stroke-width`, and `fill` live on the outer `<svg>` and the `<path>` gains `vector-effect="non-scaling-stroke"`, so the stroke renders at a constant pixel weight (1.5px) regardless of icon size. A `Stroke width` RangeControl is added to the Storybook icon-library story to visualize the system and identify which icons still need redrawing. Note that an earlier, broader version of this PR (a `strokeWidth` prop on the React `Icon` component and `wp_kses` allowlist changes in PHP) was dropped — those are deferred to follow-ups.

## Impact

- **Plugin & theme developers (JS):** No action required. The change is fully backwards compatible and opt-in per icon. Existing fill-based icons render exactly as before. There is **no** new `strokeWidth` prop on `Icon` despite what the PR description's struck-through text suggests — that was removed before merge.
- **Consumers of `square`:** No visual change at the default 24×24 size. The two existing consumers (`zoom-out-toggle`, the HTML block modal) render at 24px and are unaffected. At 16×16 or 32×32, `square` now renders a true 1.5px stroke instead of a scaled stroke.
- **PHP / server-side icon rendering:** No change in this PR. The proposed `WP_Icons_Registry::sanitize_icon_content()` allowlist additions and a `stroke_width` arg on `wp_get_icon()` are deferred and blocked on #78332.
- **Build size:** Negligible (+32 B total).

## Technical details

The substantive change is to `packages/icons/src/library/square.svg`. Stroke-related attributes are hoisted to the root `<svg>` and the path is marked non-scaling.

**Before** (stroke effectively scales with the icon's rendered size):
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
  <path stroke="currentColor" stroke-width="1.5" fill="none" d="…" />
</svg>
```

**After** (stroke pinned to 1.5px at any render size):
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"
     stroke="currentColor" stroke-width="1.5" fill="none">
  <path d="…" vector-effect="non-scaling-stroke"
        stroke-linecap="round" stroke-linejoin="round" />
</svg>
```

The mechanism is the SVG `vector-effect="non-scaling-stroke"` attribute, which decouples stroke width from the coordinate-system scaling applied when an icon drawn on a 24×24 `viewBox` is rendered at 16px or 32px. Placing `stroke`/`stroke-width`/`fill` on the outer element establishes the convention intended for the broader redraw (so a raw `.svg` previews correctly outside its rendered context).

The other change is to the Storybook icons-library story, which adds a `Stroke width` RangeControl (range 0.5–5, step 0.25). Dragging it affects stroke-based icons like `square` and is inert for fill-based icons (`plus`, `check`), making it a tool for spotting which icons still need conversion. No public JS/PHP API, hook, `block.json` field, or REST schema is changed.

## Contribution

Authored by **@jasmussen** (with Claude Code, Claude Opus 4.7, credited via `Co-Authored-By`) and merged as commit `86c5139` under labels `[Type] Enhancement`, `[Package] Icons`, `[Feature] Icons`. The PR was deliberately scoped down during review: the original proposal added a `strokeWidth` prop to the React `Icon` component (passed via `cloneElement`) and expanded the PHP `wp_kses` allowlist in `WP_Icons_Registry::sanitize_icon_content()`, but both were struck and deferred — the PHP `wp_get_icon()` `stroke_width` arg being blocked on #78332. What landed is the minimal, backwards-compatible foundation (one canary icon plus a Storybook control). The stated follow-ups are converting the remaining fill-based icons to strokes in batches and adding the PHP-side stroke support once #78332 merges.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
