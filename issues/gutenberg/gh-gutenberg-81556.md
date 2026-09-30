# #81556: Dashboard Widgets: promote high-relevance actions into a chrome footer

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`, `[Package] Widget Dashboard`
- **Merged:** [`3f15b6b`](https://github.com/WordPress/gutenberg/commit/3f15b6b8061870b9e05040b8d9c25a3973116741)
- **Discussion:** [#81556](https://github.com/WordPress/gutenberg/pull/81556) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The experimental Dashboard Widgets host in `@wordpress/widget-dashboard` now honors the `relevance` hint on widget actions. `relevance` gains a `'medium'` tier (`'high' | 'medium' | 'low'`). `'high'` actions render as leading text links in a persistent footer strip under the widget body, `'medium'` as trailing compact affordances (icon-only when the action declares an icon), and `'low'` (the default) stays in the "More" menu. Full-bleed widgets have no chrome, so all of their actions stay in the menu. The bundled `core/events` widget drops its hand-rolled footer for declared actions, and `core/quick-draft` regains a "View all drafts" link.

## Impact

**Widget authors (experimental Dashboard Widgets API)**
- `relevance: 'medium'` is now a valid value on actions in `widget.json`. The PHP sanitizer and REST schema accept it; before, anything other than `'high'`/`'low'` was dropped.
- Actions declared `'high'` or `'medium'` now leave the "More" menu and appear in the footer. If you already declared `relevance: 'high'` on actions, expect them to move.
- Footer affordances are real anchors (middle-click and copy address work). Icon-only medium actions use `aria-label` and a tooltip; `openInNewTab` adds "(opens in a new tab)" to the accessible name.
- Full-bleed widgets are unaffected: every action stays in the menu.

**Host/package consumers**
- `WidgetActions` now takes `actions: WidgetAction[]` instead of `widgetType`. This is a prop change on an internal component, so only code importing it directly is affected.

**REST consumers**
- The widget-modules schema `relevance` enum for actions is now `high`, `medium`, `low`.

**Site owners**
- Quick Draft shows a "View all drafts" link; Events keeps Meetups / WordCamps, now rendered through the shared footer. This is all under the experimental Dashboard feature.

## Technical details

**PHP / REST**
- `lib/experimental/dashboard-widgets/widget-types.php`: `gutenberg_sanitize_widget_actions()` now accepts `'medium'` in its `in_array()` check for `relevance`.
- `class-wp-rest-widget-modules-controller.php`: the action `relevance` enum in `get_item_schema()` becomes `array( 'high', 'medium', 'low' )`.

**Package (`packages/widget-dashboard`)**
- New `WidgetFooter` component (`components/widget-footer/`). It splits its actions into high and medium. High actions render as `Link`s, with a declared icon as a prefix. Medium actions sit in a trailing `Stack` (`margin-inline-start: auto`) under a `Tooltip.Provider`: an icon-only `LinkButton` via a private `IconAction` if the action has an icon, else a text `Link`. It applies `inert` when `editMode` is set.
- `IconAction` squares the `LinkButton` through the `--wp-ui-button-aspect-ratio`, `--wp-ui-button-padding-inline` and `--wp-ui-button-min-width` override hooks. For `openInNewTab` it passes `target="_blank" rel="noopener noreferrer"` on the anchor through `render`, so `Link`'s external glyph doesn't double the icon.
- `utils/split-widget-actions` (`splitWidgetActions`) returns `{ footer, menu }`. The diff shows it called from `WidgetFrame` and `Widgets`, but its source falls in the truncated part of the diff, so the exact routing rules (including the full-bleed case) aren't visible here.
- `WidgetFrame` renders `<WidgetFooter>` after `Card.Content` when `footer` is non-empty. `widgets.tsx` computes `hasActions` from the `menu` list and passes it to `WidgetActions`.
- The footer has a full-width divider (`border-block-start` using `--wpds-color-stroke-surface-neutral-weak`) and aligns content to `--wp-ui-card-padding`.
- The PR description also covers: `useWidgetTypes` holding the icon slot with a stand-in while icon references resolve, the body losing its bottom padding, picker previews inheriting the footer, the `core/events` and `core/quick-draft` changes, and a `demo/goal-progress` Storybook story. The truncated diff doesn't show these.
- Docs: the architecture doc, package README (new "Actions" section) and CHANGELOG are updated.

```diff
-if ( isset( $action['relevance'] ) && in_array( $action['relevance'], array( 'high', 'low' ), true ) ) {
+if ( isset( $action['relevance'] ) && in_array( $action['relevance'], array( 'high', 'medium', 'low' ), true ) ) {
```

## Contribution

The record shows a single-author PR from @retrofox, with review from @chihsuan credited in the props list. The author says the `'medium'` tier's materialization rule (compact trailing shape) changed after review, and docs were expanded in response to a reviewer question. Two follow-ups are deferred: a `scope` axis (`'local' | 'global'`) for reaching host-level surfaces like a command palette, and dynamic action labels carrying runtime state such as counts.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
