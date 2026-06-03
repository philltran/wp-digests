# #78572: Dashboard Widgets: Adapt Quick Draft to its tile size with a recent drafts list

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`
- **Merged:** [`9a1a88a`](https://github.com/WordPress/gutenberg/commit/9a1a88aa76d4007dd27bd0358f5dc567e4f017e3)
- **Discussion:** [#78572](https://github.com/WordPress/gutenberg/pull/78572) · 2 comments · 0 reactions

## Summary

The experimental dashboard Quick Draft widget — first introduced in #78408 — now adapts its layout to the size of its tile and surfaces the current user's recent drafts. A new `useWidgetSize` hook measures the widget's root element via `ResizeObserver` and exposes two boolean axes (`isWide`, `isTall`); the render uses those to choose between three presentations: a side-by-side form+list when wide, a stacked form-above-list when tall-and-narrow, and a compact form with an inline reveal link when small. A new `DraftsList` component — built on the DataViews list layout — shows each draft's featured image, relative date, and a trash action, and only mounts (and therefore only fires its REST request) when the layout calls for it.

## Impact

**Plugin and theme developers**
- No breaking changes. This is an additive change confined to the experimental dashboard. No public PHP or JavaScript APIs were removed or deprecated.
- Developers building their own dashboard widgets for the experimental dashboard should note the `content-bleed` presentation class and the `widget-chrome` min-size fix as patterns to follow for full-bleed list content.

**Site owners / editors**
- Quick Draft in the experimental dashboard now shows recent draft posts without leaving the dashboard; the list is only visible when the tile has room for it.
- No configuration required.

**All others**
- No action required. The classic dashboard's Quick Draft widget is unaffected.

## Technical details

**`useWidgetSize` hook**
Attaches a `ResizeObserver` to the widget's own root element and returns `{ isWide: boolean, isTall: boolean }`. The hook is purely measurement; all conditional rendering is derived from those booleans in the widget component.

**`DraftsList` component**
Uses the DataViews list layout to render recent drafts. Each row shows the post's featured image thumbnail, title, and a relative date. A trash action is available per item. Because the component is only rendered when `isWide || isTall` (or after the user clicks the reveal link), the underlying `getEntityRecords` REST fetch for `status=draft` stays dormant on small tiles — no wasted requests.

**Layout branching**
```jsx
// Pseudocode of the three render paths:
if ( isWide ) {
  // DraftsList beside the form (flex row)
} else if ( isTall ) {
  // DraftsList below the form (flex column)
} else {
  // Compact: form + "Draft posts" toggle link
  // Clicking the link mounts DraftsList in-place; "Back" unmounts it
}
```

**`content-bleed` opt-in**
The widget registers the `content-bleed` presentation so the drafts list can reach the card edges without internal padding interruption.

**`widget-chrome` CSS fix**
The chrome container's content area gains `min-block-size: 0` (and/or equivalent min sizes) to prevent the card from growing unboundedly on tall tiles. This is a shared primitive fix that benefits any widget placing scrollable content inside a constrained tile.

**Per-widget `package.json`**
A `package.json` is added to the Quick Draft widget directory, registering it as a workspace member. This aligns it with the sibling widgets in the monorepo and allows it to declare its own dependency graph (including DataViews).

## Contribution

Opened and merged by @retrofox (merged as commit `9a1a88a`). The PR builds directly on the foundation laid in #78408. @simison and @jameskoster are credited as co-authors. Formal PR discussion was thin — two automated bot comments — with no recorded design debate about alternative approaches.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
