# #81740: Dashboard Widgets: let the host decide which component renders an action link

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`, `[Package] Widget primitives`, `[Package] Widget Dashboard`
- **Merged:** [`a88863a`](https://github.com/WordPress/gutenberg/commit/a88863a56cb9761e8ab65060c2dd6f459fdfdb08)
- **Discussion:** [#81740](https://github.com/WordPress/gutenberg/pull/81740) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a host seam to `@wordpress/widget-primitives` (`WidgetHostProvider` / `useWidgetHost`) whose first capability, `links`, lets the embedding application decide how a widget action link renders. When an action href matches one of the host's own routes, the widget chrome (actions menu and footer) mounts the host router's link and navigates client-side. All other actions, including `download` and `openInNewTab`, keep the plain anchor. The Dashboard (Beta) provides the capability, so the site-health `Details` action now navigates without a full page reload and with no manifest change.

## Impact

- **Widget authors:** No change. Widget declarations remain portable data with plain hrefs such as `admin.php?page=…&p=/reports`; the host upgrades matching ones.
- **Dashboard host/application developers:** You can now wrap the dashboard in `WidgetHostProvider` with `{ links: { match, Link } }` to get client-side navigation for your own routes. `Link` must render a real anchor and accept a `path` prop. Without a provider, behavior is unchanged (plain anchors).
- **End users of Dashboard (Beta):** `Details` in the Site Health widget footer navigates client-side. Middle-click and copy-address still work because it remains a real anchor.
- **Stability:** Both packages are `0.x` and experimental, so this API may still change. Discussion in the PR indicates follow-ups are planned, including a single-copy contract (#82063) and a widget-body consumer of the context.
- No breaking changes or deprecations.

## Technical details

**New seam** (`packages/widget-primitives/src/widget-host/`): `WidgetHostProvider` and `useWidgetHost`. `WidgetHost` is a bag of optional capabilities, and the provider merges over the inherited value so capabilities can layer per subtree. The `links` capability (`WidgetHostLinks`) carries `match( href ): string | null` and a `Link` component taking `{ path }` plus anchor props. The packages treat the matched path as an opaque token (a field renamed from `to` to `path` to keep router vocabulary out of the contract).

**Chrome** (`packages/widget-dashboard`): new `getActionRoute( links, action )` in `components/widget-actions/get-action-route.ts` returns `null` when there is no `links` capability or when `action.download` / `action.openInNewTab` is set. Otherwise it returns `links.match( action.href )`. `WidgetActions` (More menu) and `WidgetFooter` (high-relevance links, compact links, and `IconAction`) call it. On a match they pass the host link through the existing `render` prop of the `@wordpress/ui` `Link` / `LinkButton` primitives, so styling is unchanged. In the menu, the `Link` is the `render` target of `Menu.Item`, so the host `Link` must forward refs.

```tsx
<WidgetHostProvider
	value={ { links: { match: matchDashboardHref, Link: AppRouterLink } } }
>
	<WidgetDashboard />
</WidgetHostProvider>
```

**Dashboard route:** `DashboardWidgetHostProvider` wraps the stage in `routes/dashboard`, and `matchDashboardHref` matches hrefs with the same document and same `page` query arg, resolving the `p` param as the route. Per a later review round, `p` accepts only root-relative paths. `routes/dashboard` gains a `@wordpress/route` dependency (`package-lock.json` updated); `widget-primitives` and `widget-dashboard` do not depend on any router.

**Other:** new `HostLinks` Storybook story with a demo host (route link, external anchor, download side by side), a Widget Host docs page, CHANGELOG and architecture-doc updates, and unit tests for the seam and the matcher. Bundle size of `widget-primitives/index.min.js` grows by about 110 B.

## Contribution

The PR was merged while @ciampo's review threads were still open. His first question was whether to follow the admin-ui Navigation pattern of a consumer-supplied `linkComponent`. @retrofox replied that `Link` is the same idea, but `match` is needed because widget hrefs come from portable definitions and something must decide which are in-app routes, and that context avoids threading routing through the dashboard's public API. @ciampo later objected to adding context and APIs not yet needed and to the merge timing. @retrofox acknowledged merging with threads unresolved and said he would ask first next time. He noted that the API-shape discussion continues and that follow-ups (#82063, the single-copy peer-dependency/duplicate-copy tests, and the widget-body consumer) would land separately. @chihsuan's second-pass review led to restricting `p` to root-relative paths and to a keyboard test and docs for the ref requirement.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
