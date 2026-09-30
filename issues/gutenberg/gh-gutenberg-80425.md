# #80425: Breadcrumb: Add UI component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`8df2c83`](https://github.com/WordPress/gutenberg/commit/8df2c834e05a14df90bc8906afd24774d653ed6e)
- **Discussion:** [#80425](https://github.com/WordPress/gutenberg/pull/80425) · 9 comments · 3 reactions
- **Usefulness:** 3/5

## Summary

Adds a responsive `Breadcrumb` compound component to `@wordpress/ui`, exposed as `Breadcrumb.Root`, `Breadcrumb.LinkItem`, and `Breadcrumb.CurrentItem`. Ancestor links collapse into an overflow `Menu` when space runs out, while the current item stays visible. The component is routing-agnostic: router-aware links are composed through the standard `render` prop. It also supports truncated-label tooltips, RTL, and focus preservation while resizing.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:** A new breadcrumb primitive is available, noted in the package CHANGELOG under "New Features". No existing API changes, so no action is required.
- **Admin UI / Boot work:** The discussion says the component is not yet used anywhere and that coordination on adopting it in admin UI is the next step.
- **Site owners, hosts, and REST consumers:** Not affected.
- **Bundle size:** The compressed-size report shows +3.25 kB overall across the tracked build outputs. The diff itself only adds files under `packages/ui` and does not show why the size changed across multiple script bundles.

## Technical details

The diff (truncated in the provided input) adds a `packages/ui/src/breadcrumb/` module. Its `index.ts` exports `CurrentItem`, `LinkItem`, and `Root`.

- **Render modes:** A `BreadcrumbItemRenderContext` (`context.tsx`) carries `mode: 'measurement' | 'overflow' | 'visible'` plus `itemKey`, focus callbacks, separator/measurement refs, and `shouldTruncateCurrent`. `useBreadcrumbItemRenderContext()` throws in non-production builds if items are not rendered as direct children of `Breadcrumb.Root`.
- **Item components:**
  - `LinkItem` renders a `Link` (`tone="neutral"`) in visible mode, a `Menu.LinkItem` with `closeOnClick` in overflow mode, and an inert copy in measurement mode.
  - `CurrentItem` renders a `span` with `aria-current="page"`.
  - Both wrap their content in `Tooltip.Root`, disabled unless `useIsTruncated` reports truncation.
  - A truncated current item gets `tabIndex=0` so it is keyboard-focusable.
- **Layout algorithm:** `layout.ts` exports `getCollapsedLayout(metrics, pinnedIndex?)` with a 1px `FIT_TOLERANCE`. If everything fits, nothing collapses. Otherwise, in order of priority, it keeps:
  1. the focused (pinned) link, if it fits;
  2. the first link;
  3. as many trailing links as fit.

  The rest go to `collapsedIndices`. It sets `shouldTruncateCurrent` when the overflow trigger plus the current item cannot fit. If the pinned item cannot fit, focus is moved to the overflow trigger (`shouldMoveFocusToOverflow`).
- **Custom renderers:** `enforce-render-props.ts` clones `render` elements or functions to force props such as `aria-current` and `href`. `measurement-render.ts` strips behavior props (`id`, `ref`, `tabIndex`, `on*` handlers) from the measurement copy so it mirrors the visible tree without side effects.
- **Styling:** Uses `global-css-defense.module.css` (extended to `ol` and `li` per the discussion), `resets.module.css`, and `focus.module.scss`.
- **Tooling:** `@wordpress/jest-console` is added as a devDependency of `@wordpress/ui` (`package.json`, `package-lock.json`) and imported in `global.d.ts`.

Usage follows the compound pattern:

```jsx
<Breadcrumb.Root>
  <Breadcrumb.LinkItem href="/">Home</Breadcrumb.LinkItem>
  <Breadcrumb.LinkItem href="/posts">Posts</Breadcrumb.LinkItem>
  <Breadcrumb.CurrentItem>Hello world</Breadcrumb.CurrentItem>
</Breadcrumb.Root>
```

The `Root` implementation and the overflow trigger button file are cut off in the provided diff, so their details are not covered here. The usage snippet is illustrative, based on the API names in the PR description.

## Contribution

Authored by @ciampo, closing the tracking issue #77039; the initial specs were co-authored by @grbicsanja in #76709. During review, feedback on `common.css` style leaks led to extending the global CSS defense to `ol` and `li`. The author also fixed focus-management bugs around resizing and improved intrinsic measurement to account for the open-in-new indicator and custom renderers. A concern about highlighting a selected item inside the collapsed menu was dropped, since the current page never collapses by design. The PR was merged with a note to iterate later, and the author planned to coordinate adoption in admin UI with @simison. The PR disclosed that Codex assisted with implementation, tests, and self-review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
