# #82952: Widget Primitives: add `HostLink` over the `links` capability

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`, `[Package] Widget primitives`, `[Package] Widget Dashboard`
- **Merged:** [`3d14252`](https://github.com/WordPress/gutenberg/commit/3d14252b5ed2d64b4d1760d2083f9a6e6ecad999)
- **Discussion:** [#82952](https://github.com/WordPress/gutenberg/pull/82952) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/widget-primitives` now exports a `HostLink` component that encapsulates the decision of whether to mount the host router's `Link` (when `links.match` recognizes the target) or render a plain `<a>` anchor. The widget dashboard's actions menu, footer, and the Site Health widget body previously each repeated that branching logic by calling `useWidgetHost()` and the now-removed `getActionRoute` helper; they all render through `HostLink` via the `render` prop of their respective UI link components. Behavior is unchanged — the PR is a consolidation that also makes the routing decision available to any widget dashboard consumer.

## Impact

- **Widget dashboard / widget primitive developers:** A new public export `HostLink` is available from `@wordpress/widget-primitives`. If you build custom dashboard chrome or widget bodies that need to render a link target, you can now pass `href` (plus standard anchor props) to `HostLink` inside a UI link's `render` prop instead of manually calling `useWidgetHost()` and branching on `links.match`. No migration is required for existing code — the dashboard internals were updated in this PR.
- **Plugin & theme developers (general):** No action required. The change is internal to the widget dashboard packages and does not alter any public REST, block, or template API.
- **Hosting & platform:** No action required. Bundle size increases by 163 B in `build/modules/widget-primitives/index.min.js`.

## Technical details

**New component** — `packages/widget-primitives/src/widget-host/host-link.tsx` (file path inferred from the PR description; the diff is truncated before this file's contents):

- Accepts `href` plus standard anchor props (`download`, `target`, `rel`, etc.) and forwards its `ref`.
- Calls `links.match(href)` from the `WidgetHostLinks` capability. If it returns a non-null path, it mounts `links.Link` with that path; otherwise it renders a plain `<a>`.
- A target that opens a new document never routes: a `download` value other than `false`, or a `target="_blank"`, forces the plain-anchor branch.
- Its props interface is local to the file (not re-exported), so the package's public surface gains only the `HostLink` name.

**Removed** — `packages/widget-dashboard/src/components/widget-actions/get-action-route.ts` and its test `packages/widget-dashboard/src/test/get-action-route.test.ts`. The routing rule (check `download`, `openInNewTab`, then `links.match`) moved into `HostLink`; the test cases moved into `host-link.jsdom.test.tsx`.

**Dashboard chrome refactored** — In `widget-actions.tsx`, the per-action `getActionRoute` call and the `linkProps` ternary are replaced by a single `render={ <HostLink href={ action.href } /> }` on `Menu.LinkItem`, with `download` and `openInNewTab` passed as direct props. In `widget-footer.tsx`, the three separate branches (high-priority `Link`, medium `Link`, and `IconAction`) each collapse to one shape that passes `render={ <HostLink href={ action.href } /> }`. The `routeRender` prop is removed from `IconAction`; the new-tab `target`/`rel` now ride on the `HostLink` anchor rather than on the UI `LinkButton`, whose own glyph would otherwise duplicate the icon.

**Site Health body** — `widgets/site-health/render.tsx` renders its review link through `HostLink`, so the widget body no longer reads the `links` capability directly.

**Story** — A `WithHostLink` story in `packages/widget-primitives/src/components/widget-render/stories/index.story.tsx` shows three action targets against a demo router, plus a button that removes `links` from the host bag to demonstrate the plain-anchor fallback.

**Docs** — `packages/widget-primitives/README.md` restructures the host sections so `HostLink` leads, ahead of `WidgetHostProvider` / `useWidgetHost`.

## Contribution

Opened by @retrofox as a follow-up to a series of widget-primitives PRs (#81740, #82066, #81929) and part of the broader #79231 epic. Co-authored with @chihsuan. The PR closed #82068. The record carries no substantive design debate — the two comments are the standard GitHub Actions bot posts (contributor attribution and PR meta with bundle-size/performance data).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
