# #82684: Block Editor: Update LinkControl and LinkPicker to use `Badge` from `@wordpress/ui`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @hbhalodia
- **Labels:** `[Type] Enhancement`, `[Feature] UI Components`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`b0975ae`](https://github.com/WordPress/gutenberg/commit/b0975aebdede5c22d1c44c96fd620f35e0aad624)
- **Discussion:** [#82684](https://github.com/WordPress/gutenberg/pull/82684) · 12 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `LinkControl` and `LinkPicker` components in `@wordpress/block-editor` now render their link-preview badges using the public `@wordpress/ui` `Badge` instead of the private `@wordpress/components` `Badge` (previously accessed via `unlock(componentsPrivateApis)`). As part of the migration, the `computeBadges` helper in `@wordpress/block-library` (Navigation Link) now emits the new `@wordpress/ui` intent vocabulary (`none`, `low`, `informational`, `stable`, `high`) in place of the old `@wordpress/components` vocabulary (`default`, `warning`, `success`, `error`). This is one step in the broader deprecation of the `@wordpress/components` `Badge` component (tracked in #82440).

## Impact

- **Plugin developers using `LinkPicker` with a custom `preview` prop:** If you pass `badges` with the old intent strings (`default`, `warning`, `success`, `error`), they will no longer map to the correct visual treatment. Update to the new vocabulary: `default` → `none`, `warning` → `low` or `informational`, `success` → `stable`, `error` → `high`.
- **Plugin developers calling `computeBadges` from `@wordpress/block-library`:** The returned badge objects now carry the new intent values. Any downstream rendering or conditional logic keyed on the old strings will break.
- **Theme / site owners:** No action required. The visual change is a minor restyling of the small status badges (Draft, Published, External link, etc.) shown in the link preview panel.
- **No action required** for plugins that only consume `LinkControl` without supplying custom badge data, since the badges are generated internally.

## Technical details

Two files in `@wordpress/block-editor` swap the import and JSX tag:

```jsx
// Before (link-control/link-preview.jsx, link-picker/link-preview.jsx)
import { privateApis as componentsPrivateApis } from '@wordpress/components';
import { unlock } from '../../lock-unlock';
const { Badge: WCBadge } = unlock( componentsPrivateApis );
// …
<WCBadge intent={ badge.intent }>{ badge.label }</WCBadge>

// After
import { Badge } from '@wordpress/ui';
// …
<Badge intent={ badge.intent }>{ badge.label }</Badge>
```

The substantive behavioral change is in `packages/block-library/src/navigation-link/shared/use-link-preview.js`, where `computeBadges` maps entity status and link type to the new intent vocabulary:

| Badge label | Old intent | New intent |
|---|---|---|
| External link / Internal link / Homepage / entity type / Page | `default` | `none` |
| No link selected / Missing %s / Trashed | `error` | `high` |
| Published | `success` | `stable` |
| Draft / Pending | `warning` | `low` |
| Scheduled | `warning` | `informational` |
| Private | `default` | `informational` |

The `trash` status label was also renamed from `Trash` to `Trashed` for consistency. The JSDoc for `computeBadges` was updated to reference `@wordpress/ui` `Badge` intent. All corresponding jsdom tests in `link-control/test/index.jsdom.test.jsx`, `link-picker/test/index.jsdom.test.jsx`, and `navigation-link/shared/test/use-link-preview.jsdom.test.js` were updated to assert the new intent strings.

## Contribution

Opened by @hbhalodia as part of the #82440 migration effort. @shail-mehta initially suggested using `none` as the default intent rather than `draft`; @hbhalodia had originally chosen `draft` to preserve the grey background of the old `default` intent but agreed to switch after @simison and @jasmussen confirmed the design direction. @fcoveram confirmed that Internal/External link badges should also use `none`. The `Trash` → `Trashed` label change and the `Scheduled` → `informational` mapping (rather than `low` like Draft) were settled during review. Merged as `b0975ae`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
