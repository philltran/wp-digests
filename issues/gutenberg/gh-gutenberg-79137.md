# #79137: Global styles revisions: replace active text with badge

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `Global Styles`
- **Merged:** [`556eb97`](https://github.com/WordPress/gutenberg/commit/556eb97fec7da5d6a10819950f2c38edfd8733f9)
- **Discussion:** [#79137](https://github.com/WordPress/gutenberg/pull/79137) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Global Styles UI now replaces the verbose "These styles are already applied to your site." paragraph with a compact "Active" badge in the revisions list. This reduces visual clutter and saves vertical space in the Site Editor's revisions panel. The change updates how the active revision state is visually communicated without altering underlying revision logic or data structures.

## Impact

- **Theme & plugin developers:** Custom CSS targeting `.global-styles-ui-screen-revisions__applied-text` will break and must be migrated to `.global-styles-ui-screen-revisions__active-badge`.
- **Site editors / content creators:** The revisions panel will display a smaller badge indicator instead of a full sentence when a revision matches the currently saved styles.
- **No action required** for standard installations or plugins that do not override Global Styles UI styles.

## Technical details

The change modifies `packages/global-styles-ui/src/screen-revisions/revisions-buttons.tsx` to swap a `<p>` element for a `Badge` component accessed via the private API unlock pattern:

```js
import { privateApis as componentsPrivateApis } from '@wordpress/components';
import { unlock } from '../lock-unlock';

const { Badge: WCBadge } = unlock( componentsPrivateApis );
```

In the JSX, the previous paragraph is replaced with:

```jsx
<WCBadge
    className="global-styles-ui-screen-revisions__active-badge"
    intent="info"
>
    { __( 'Active' ) }
</WCBadge>
```

Corresponding SCSS in `packages/global-styles-ui/src/screen-revisions/style.scss` removes `.global-styles-ui-screen-revisions__applied-text` from existing selectors and applies the updated margin and alignment rules to `.global-styles-ui-screen-revisions__active-badge`. The margin is tightened from `$grid-unit-05 $grid-unit-15 $grid-unit-15 $grid-unit-50` to `0 $grid-unit-15 $grid-unit-15 $grid-unit-50` to accommodate the badge's compact height.

## Contribution

Opened and merged by @ramonjd, with the initial concept credited to @talldan. During review, @talldan noted the change was intended as an interim UI tweak ahead of a planned DataViews migration for the revisions picker, but both contributors agreed the small scope justified shipping it immediately. The PR moved quickly through review with no significant design debates or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
