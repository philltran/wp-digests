# #82457: Site editor v2: template parts take view config from endpoint

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @oandregal
- **Labels:** `[Type] Enhancement`, `[Feature] Site Editor`, `[Package] Views`
- **Merged:** [`421cb3c`](https://github.com/WordPress/gutenberg/commit/421cb3cffca3a57caecc3d759c812552999d3118)
- **Discussion:** [#82457](https://github.com/WordPress/gutenberg/pull/82457) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The experimental Extensible Site Editor (site-editor-v2) Template Parts list screen now reads its view configuration from `GET /wp/v2/view-config?kind=postType&name=wp_template_part` instead of hard-coded client defaults. Tabs, default view, and available layouts come from the server, so they follow the areas the active theme registers and honor the `get_entity_view_config_posttype_wp_template_part` filter. The PR also fixes a `@wordpress/views` bug where a persisted layout `type` that a screen no longer offers left the DataViews screen blank with no way to reset it.

## Impact

- **Plugin & theme developers:** On the site-editor-v2 Template Parts screen, you can change the default view, layouts, and tab list by filtering `get_entity_view_config_posttype_wp_template_part`. A theme that registers no sidebar area gets no "Sidebar" tab.
- **URL/route consumers:** Tab slugs in the URL are now the server-provided ones (`all-parts`, `header`, `footer`, ...), and `/template-parts` redirects to `/template-parts/list/all-parts`. Any bookmarks, tests, or code relying on the old `all` slug will not match a view entry.
- **`@wordpress/views` consumers:** `useView` and `loadView` now ignore a persisted `type` that `defaultLayouts` does not offer. Such a type no longer counts as `isModified` and is dropped from the stored preference on the next `updateView`. Without `defaultLayouts`, every type is still honored.
- **Site owners:** This is behind the Extensible Site Editor experiment in Gutenberg → Experiments. There is no effect on the stable site editor or on core.
- No PHP changes; the `wp_template_part` config already ships in core.

## Technical details

**Template parts route (`routes/template-part-list/`)**
- `stage.tsx` splits into `TemplatePartList` and `TemplatePartListView`. The outer component calls `useViewConfig( { kind: 'postType', name: 'wp_template_part' } )` and destructures `default_view`, `default_layouts`, and `view_list`. It renders `null` if `default_view` is missing.
- `activeViewOverrides` is taken from the `view_list` entry whose `slug` matches the `$area` route param.
- Tabs are built from `viewList` using `entry.slug` and `entry.title`. The `DEFAULT_VIEW`, `DEFAULT_VIEWS`, and `getActiveViewOverridesForTab` client constants are removed from use.
- The locked area is derived via a new `getAreaFromViewOverrides( activeViewOverrides )` helper in `view-utils.ts` (reading the locked `area` filter). The Area column is hidden when that value is set, replacing the old `area !== 'all'` check.
- `defaultLayouts` is passed to both `useView` and `DataViews`, so only the server-declared layouts (table, grid) are offered.
- The `'wp_template_part'` literal is centralized as `TEMPLATE_PART_POST_TYPE`. `routes/template-part/route.ts` redirects to `all-parts`.

**`packages/views`**
- New exported `getApplicablePersistedView( persistedView, defaultLayouts )` in `resolve-view.ts`. It strips `type` from the persisted overrides when `defaultLayouts` is defined and has no truthy entry for that type, returning `undefined` if nothing else remains.
- `resolveView` and `getUserModifications` both use it. `useView` computes `isModified` from the applicable view.
- `updateView` still compares new modifications against the preference *as stored*, so a stale `type` is written out on the first update, even one that changes nothing (which clears the preference).

```ts
// Before: persisted { type: 'list' } with defaultLayouts { table, grid }
// -> view.type = 'list', DataViews renders nothing, isModified = true
// After:
// -> view.type resolves from layers below ('table'), isModified = false
```

Tests were added in `src/test/resolve-view.ts` and `use-view.jsdom.test.tsx`. The diff was truncated, so the remaining `view-utils.ts` changes and e2e spec updates are not visible here.

## Contribution

Authored by @oandregal as part of the extensible site editor work tracked in #79895, and implemented with Claude Code per the PR's AI disclosure. The PR has one review bot pass (CodeRabbit, no actionable comments) and no human review discussion recorded. The `@wordpress/views` persisted-type fix appears to have been added during this PR after the layout restriction exposed the blank-screen case.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
