# #81729: Dashboard: add a Site Health detail page as the dashboard's second route

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] DataViews`, `[Feature] Dashboard`
- **Merged:** [`6e4cf9c`](https://github.com/WordPress/gutenberg/commit/6e4cf9c2925651a1d2087bed4e54db49ee0ac043)
- **Discussion:** [#81729](https://github.com/WordPress/gutenberg/pull/81729) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `routes/site-health`, the second route in Gutenberg's experimental Dashboard app, giving the site-health widget an in-app destination instead of only linking out to the classic Site Health screen. The page fetches the six async health tests core exposes over the REST namespace `wp-site-health/v1` and renders them in a DataViews table with sortable/filterable Status and Category badge columns and a per-check description. A new `Details` action in `widgets/site-health/widget.json` points at `admin.php?page=dashboard-wp-admin&p=/site-health` — still a plain anchor, so following it is a full page load until the router-link upgrade lands (#81725). The widget's own fetch list also picks up the previously missing `page-cache` test, and the dashboard route swaps its `Page` `title` prop for a `Breadcrumbs` element.

## Impact

- **Plugin & theme developers:** No action required — everything here is behind the *New Dashboard experience* experiment in Gutenberg → Experiments. The interesting part is the convention it demonstrates: a route is a workspace package under `routes/` whose `package.json` carries a `route` manifest (`{ "path": "/site-health", "page": [ "dashboard" ] }`), picked up by the build registry with no manual registration, plus a `route.ts` exporting a `title` and a `stage.tsx` exporting `stage`.
- **Anyone building dashboard widgets:** `widget.json` `actions[]` entries accept an `href` targeting an in-app route via the `p=` query arg (`admin.php?page=dashboard-wp-admin&p=/site-health`), with `relevance: "high"` promoting the action into the widget footer.
- **Site owners:** The new page covers only the six *async* checks reachable over REST. Direct (PHP-only) checks have no endpoint and remain exclusive to the classic Site Health screen — the new page is not a replacement.
- **Behavior change in the existing widget:** `widgets/site-health/render.tsx` now also requests `/wp-site-health/v1/tests/page-cache`, so widget issue counts will include page-cache results that were previously omitted.
- **No public API, hook, filter, REST route, or database change** — this consumes existing core endpoints only.

## Technical details

**New package** `@wordpress/site-health-route` at `routes/site-health/`, wired into the workspace via `package-lock.json` and depending on `@wordpress/admin-ui`, `api-fetch`, `dataviews`, `dom`, `element`, `i18n`, and `@wordpress/ui`. `route.ts` is minimal:

```ts
export const route = {
	title: () => __( 'Site Health' ),
};
```

**Data fetching** (`routes/site-health/stage.tsx`): a `useEffect` runs `Promise.allSettled` over a frozen `ASYNC_TEST_PATHS` list of six `apiFetch` calls — `background-updates`, `loopback-requests`, `https-status`, `dotorg-communication`, `authorization-header`, `page-cache` under `/wp-site-health/v1/tests/`. A `cancelled` flag guards the async setState. Each fulfilled response goes through `normalizeCheck()`, which requires a non-empty string `label` and a `status` in `[ 'critical', 'recommended', 'good' ]`, optionally reads `badge.label` as the category, and converts `description` HTML to lines; anything else returns `null` and increments a `dropped` counter. Rejections contribute a message via `toMessage()` (shape-tolerant read of `reason.message`, run through `__unstableStripHTML`).

`descriptionToLines()` splits the description on `</p>` and `</li>`, strips HTML, collapses whitespace, and drops empties, so paragraphs and result items each get their own line rendered with `<br />` separators.

**Rendering:** `DataViews` + `filterSortAndPaginate` with a default view of `{ type: 'table', perPage: 10, titleField: 'label', descriptionField: 'description', fields: [ 'status', 'category' ], sort: { field: 'status', direction: 'asc' } }`. Status and Category render as `Badge` from `@wordpress/ui`; status intent is mapped `critical → high`, `recommended → medium`, `good → stable`, and a custom `sort` comparator orders by `STATUS_ORDER` (`critical` 0, `recommended` 1, `good` 2) — with an inline note that DataViews passes field *values* to the comparator despite the public type declaring items, because `normalizeFields` wraps it over `getValue`. Category `elements` are derived from the fetched data, so the filter only offers categories actually present.

**States:** when every check failed (`checks.length === 0 && unavailable > 0`) the page renders an error `Page` with either `'The site health checks could not run: %s'` or the generic variant; a partial failure instead shows an `_n()` inline note (`'%d check could not run. Reload the page to try again.'`) above the table.

**Widget action** (`widgets/site-health/widget.json`), inserted ahead of the existing `site-health-status` action:

```json
{
	"id": "site-health-details",
	"label": "Details",
	"icon": "core/chart-bar",
	"relevance": "high",
	"href": "admin.php?page=dashboard-wp-admin&p=/site-health"
}
```

**Dashboard chrome** (`routes/dashboard/stage.tsx`) moves off the `Page` `title` prop:

```jsx
// Before
<Page title={ editMode && isMobileViewport ? undefined : pageTitle } … />

// After
<Page
	breadcrumbs={
		editMode && isMobileViewport ? undefined : (
			<Breadcrumbs items={ [ { label: pageTitle } ] } />
		)
	}
	…
/>
```

The Site Health page uses the two-level form, `[ { label: 'Dashboard', to: '/' }, { label: 'Site Health' } ]`. A `tsconfig.json` extending `tsconfig.base.json` (`noEmit`, `jsx: react-jsx`, `composite: false`) is added for the new package. Both the route and the widget carry paired comments instructing that the two `ASYNC_TEST_PATHS` lists be kept in sync — the duplication is deliberate and unenforced.

## Contribution

The thread carries no human review discussion — the only comments are bots (a 0 B bundle-size report, an unrelated flaky navigation-frontend-interactivity e2e failure, and props-bot). Props-bot credited @chihsuan alongside the author. The PR closes #81724 as part of the larger dashboard effort #81723, and explicitly defers client-side navigation for the new `Details` link to follow-up #81725, shipping the anchor as a full page load in the meantime.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
