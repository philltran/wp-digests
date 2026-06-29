# #79517: Widget inserter: more accurate widget previews

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`
- **Merged:** [`1770b8d`](https://github.com/WordPress/gutenberg/commit/1770b8d8bb71fa760b763d5597b2afdf011056ac)
- **Discussion:** [#79517](https://github.com/WordPress/gutenberg/pull/79517) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The widget dashboard inserter now renders accurate, framed previews of each widget instead of just scaling down raw markup. Previously, the picker cached unstyled widget bodies and applied a hardcoded `transform: scale(0.8)`, producing misaligned or truncated visuals. The update extracts shared rendering logic into a dedicated frame component so catalog previews visually match the final dashboard tile, including headers, icons, and proper spacing. Picker tile sizing also increased from 230px to 290px for better readability.

## Impact

['- **Dashboard users / site owners**: Widget picker previews now display full card chrome (headers, loading/error states) scaled cleanly to fill tiles, matching the final layout before insertion.', '- **Plugin & theme developers**: No action required. The change is confined to internal dashboard package components (`@wordpress/widget-dashboard`) and does not alter public widget registration APIs, block patterns, or REST endpoints.', '- **Hosting & platform teams**: No configuration, migration, or caching changes needed.']

## Technical details

The diff refactors `packages/widget-dashboard/src/components/widget-chrome/widget-chrome.tsx` to extract header/content/error-boundary logic into a new shared component: `WidgetFrame`. A new `packages/widget-dashboard/src/components/widget-preview-chrome/` module hosts `WidgetFrame` inside an inert, click-transparent grid viewport, replacing the previous approach in `widget-picker.tsx` where `<WidgetRender>` was wrapped in a scaling wrapper div. Legacy CSS rules (`widgetChrome`, `widgetChromeContentBleed`, `widgetChromeBleedScroll`) were removed from `widget-chrome.module.css` as layout containment and internal scrolling are now handled consistently by the frame. The picker's default grid configuration was updated to set `layout.previewSize: 290`. The dashboard's primary rendering path remains untouched; only the insertion catalog receives the frame-based preview.

## Contribution

Opened by @retrofox and merged following review from @simison. The discussion acknowledged several follow-up opportunities (e.g., reducing tile scale to prevent crowding, pre-populating preview attributes similar to block `example` definitions, and fixing drag-shadow rendering artifacts), but all were explicitly deferred as non-blocking to keep the merge focused on UI accuracy and visual polish.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
