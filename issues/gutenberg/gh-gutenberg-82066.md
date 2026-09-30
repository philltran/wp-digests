# #82066: Site Health widget: route the body's review link through the host

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Status] In Progress`, `[Feature] Dashboard`, `[Package] Widget primitives`
- **Merged:** [`3b76dad`](https://github.com/WordPress/gutenberg/commit/3b76dad973ca95665a09c0cfc8ab2ce4385adc40)
- **Discussion:** [#82066](https://github.com/WordPress/gutenberg/pull/82066) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Site Health dashboard widget's "Review N items" link now targets the dashboard's Site Health page filtered to `status=critical,recommended` (only statuses that have items) instead of the classic `site-health.php` screen. The link renders through the host `links` capability, so it navigates client-side on the dashboard and stays a plain anchor elsewhere. To support this, the Site Health route now reads and writes a `status` search param, and the dashboard href matcher accepts a route query inside `p` when both client-side and full-load navigation would read it identically.

## Impact

- **Plugin/widget authors (Gutenberg dashboard, beta):** `WidgetHostLinks.match` may now return a string carrying a query (e.g. `'/sales?by=day'`). Consumers must keep handing it back to `Link` uninterpreted. Hosts implementing their own `links` capability should be prepared for that shape. The seam type is unchanged (`string | null`).
- **Widget authors declaring action hrefs:** a `p` value with its own query (e.g. `p=%2Fsales%3Fby%3Dday`) now matches and navigates client-side, provided every key appears once and no value parses as JSON (numbers, booleans, quoted strings, objects, arrays). Otherwise it stays a plain anchor with a full load, as before.
- **Users of the Dashboard (Beta):** the widget count matches the row count on arrival, and the filter chip shows the included statuses. The `Details` action still opens the unfiltered page. Clearing the chip drops `status` from the URL.
- **Host implementers outside the dashboard:** no action required; without `links`, the link is a plain anchor to the same filtered URL.
- No action required for site owners.

## Technical details

**`widgets/site-health/render.tsx`**: builds the dashboard page href with `status=critical,recommended` (only statuses with items), the query encoded inside `p`. It calls `useWidgetHost().links` and, on a `match`, mounts the host link via `Link`'s `render` prop; otherwise it uses a plain anchor with the same href. This is the first widget body consuming `useWidgetHost()`.

**`routes/dashboard/widget-host/match-dashboard-href.ts`**: previously rejected any `p` containing `?`. Now it splits `p` at the first `?`, validates only the pathname as root-relative (`/^\/(?!\/)/`), and still rejects `#`. A new `isPlainQuery()` returns false for a repeated key or any value for which `JSON.parse` succeeds, because the router's parser would fold or coerce those on a full load but the route link would pass strings.

```
// before
match('admin.php?page=dashboard&p=%2Freports%3Fperiod%3D7d') // null
// after
match('admin.php?page=dashboard&p=%2Freports%3Fperiod%3D7d') // '/reports?period=7d'
match('...p=%2Fposts%3Fauthor%3D12')                          // null (parses as JSON number)
match('...p=%2Fposts%3Fstatus%3Ddraft%26status%3Dpending')    // null (repeated key)
```

**`dashboard-widget-host-provider.tsx`**: new `toRouteTarget()` splits the path at `?` and passes `{ to, search }` to the router `Link`, with `search` built via `Object.fromEntries(new URLSearchParams(...))` (one string per key). The description notes that using the router's own parser is the proper fix, which needs `@wordpress/route` to expose it (#82167, not touched here).

**`routes/site-health/stage.tsx`** (from the visible diff, which is truncated): adds `statusesFromSearch()` (comma-separated `status` param, unknown values dropped, deduped), imports `useNavigate`/`useSearch` from `@wordpress/route`, and per the description seeds the view with an `isAny` filter, writes chip changes back via `navigate( { search, replace: true } )`, and applies URL changes (back/forward) to the view. The status field accepts only the `isAny` operator. `@wordpress/route` is added to `routes/site-health/package.json`, and `@wordpress/widget-primitives` to the widget's dependencies (and `package-lock.json`).

**`@wordpress/widget-primitives`**: docs only. The `match` docblock, README, Storybook page, and seam SVG now describe the return as a route with its query; a CHANGELOG entry is added under Documentation. Unit tests cover the matcher (accepted comma list; rejected repeated keys, numbers, booleans, JSON, quoted strings, query-only `p`) and the provider's `search` handoff.

## Contribution

Authored by @retrofox as a follow-up to #81740 and #81729 and closing #82065; the concrete consumer of `useWidgetHost()` was requested in review on #81740, and the matcher widening was anticipated in a review thread there. The parser-based approach is deferred to #82167, so this PR splits the query manually instead of touching `@wordpress/route`. @chihsuan is credited in the props list; the visible discussion is otherwise bot output only.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
