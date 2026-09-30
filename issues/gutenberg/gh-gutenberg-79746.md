# #79746: Admin UI: Add Navigation component and Page navigation slot

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] Admin UI`
- **Merged:** [`08e4e81`](https://github.com/WordPress/gutenberg/commit/08e4e81221c8c2eb70d4500bbf0d7676fccbaa74)
- **Discussion:** [#79746](https://github.com/WordPress/gutenberg/pull/79746) · 17 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`Page` in `@wordpress/admin-ui` gains an optional `navigation` prop (`{ items, currentHref, ariaLabel }`) that renders a link-based section nav in the page header, plus a top-level `components.link` override for client-side routing. It gives screens a semantically correct alternative to `Tabs` (ARIA `tablist`) for switching between URLs/sections. The underlying `Navigation` component is internal and not exported.

## Impact

- **Plugin/theme/admin-screen developers using `@wordpress/admin-ui`:** Opt-in and non-breaking. Existing `Page` usage is unaffected. Screens that used `Tabs` to move between sections/URLs can switch to `<Page navigation>` to get real links (history, deep-linking, open-in-new-tab, `aria-current="page"`).
- **Router integration:** Default output is plain `<a>` elements, so in-app navigation causes a full page load unless you pass `components.link` (a component receiving standard anchor attributes with `href` required) to hook in a client-side router.
- **Dev-time errors:** In non-production builds, an item with a non-string `href` or a duplicate `href` throws. Passing an empty string `href` is allowed.
- **Site owners / hosting / REST consumers:** No impact.
- No migration or configuration required.

## Technical details

**New files in `packages/admin-ui/src/navigation/`:** `index.tsx`, `types.ts`, `style.module.css`, and tests.

- `Navigation` renders `<nav aria-label>` containing a `Stack` rendered as `<ul role="list">` (explicit role so Safari+VoiceOver announces the list despite `list-style: none`; the `jsx-a11y/no-redundant-roles` rule is disabled inline). Each `<li>` holds a `Text` rendering a `@wordpress/ui` `Link` with `variant="unstyled"`, which in turn renders `linkComponent ?? 'a'` with `href`. The item whose `href === currentHref` gets `aria-current="page"`.
- Returns `null` when `items` is empty. `ariaLabel` defaults to `__( 'Sections' )`.
- Under `process.env.NODE_ENV !== 'production'` it throws on a non-string `href` or duplicate `href`.
- CSS: the list scrolls horizontally (`overflow-x: auto`) instead of wrapping, with padding/negative margin/`scroll-padding` of `2 * var(--wpds-border-width-focus)` so the focus ring isn't clipped. Items have a 44px `min-block-size`/`min-inline-size`; the current item uses a stronger `--wpds-color-foreground-interactive-neutral-weak-active` color and emphasis font weight, with no underline.

**Types:**

```ts
interface NavigationItem { label: string; href: string }
interface NavigationConfig {
  items: readonly NavigationItem[];
  currentHref?: string;
  ariaLabel?: string; // default 'Sections'
}
// internal: NavigationProps extends NavigationConfig { linkComponent?; className? }
```

`NavigationLinkProps` is anchor attributes with `href` required.

**`Page` changes:** `page/index.tsx` accepts `navigation?: NavigationConfig` and `components?: PageComponents` (type imported from `./types`; that file's contents fall in the truncated part of the diff). The header now renders when `navigation.items.length` is truthy, even without title/actions/etc. `page/header.tsx` renders `<Navigation {...navigation} linkComponent={components?.link} />` below the subtitle and adds a `has-navigation` class. `page/style.module.css` sets `padding-block-end: 0` on the header (and subtitle) when navigation is present.

**Usage:**

```jsx
<Page
  title={ __( 'Analytics' ) }
  components={ { link: ( props ) => <MyRouterLink { ...props } /> } }
  navigation={ {
    items: [
      { label: 'Overview', href: '/overview' },
      { label: 'Products', href: '/products' },
    ],
    currentHref: '/overview',
  } }
/>
```

**Stories:** `withRouter` moves from a meta-level decorator to per-story decorators on the breadcrumb stories and `FullHeader`; new stories `WithNavigation`, `WithInteractiveNavigation`, and `WithNavigationAndActions` were added. A CHANGELOG entry was added under Unreleased. The `Navigation` component is not exported; only the `Page` props are public.

## Contribution

The PR was first moved to draft after @ciampo asked to coordinate with the `@wordpress/ui` `NavigationMenu` plan (#79154; Base UI-based, pending design specs and the `Menu` component in #79560). @aduth then proposed a narrower path: a configuration-based `navigation` prop on `Page` limited to label, URL and selected state, with an unexported internal component and no `@wordpress/route` coupling (per #77040). He suggested `href`/`selected` naming over `to`/`active`. @ciampo suggested passing the selected `href` separately to keep the items array stable across renders, and @retrofox combined these into a single `{ items, currentHref }` object. The PR also included the `components.link` injection to allow client-side routing without a router dependency. It is explicitly a short-term step, with the general `NavigationMenu` and migration of `Tabs`-as-navigation left as follow-ups.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
