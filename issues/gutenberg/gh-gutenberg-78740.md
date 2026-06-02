# #78740: Dashboard: Use Howdy greeting for page title

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`
- **Merged:** [`1a04ac9`](https://github.com/WordPress/gutenberg/commit/1a04ac94d807521f6fb496ce00860ea02a2f4441)
- **Discussion:** [#78740](https://github.com/WordPress/gutenberg/pull/78740) · 2 comments · 1 reactions

## Summary

The new Gutenberg-powered dashboard now displays `Howdy, {name}` as its page title instead of the static string `Dashboard`. The greeting name is resolved from `@wordpress/core-data`'s `getCurrentUser()` selector using a priority chain: trimmed display name (`name`) → `username` → `slug`. During Customize mode the title remains `Customize Dashboard` (unchanged), and `Dashboard` is shown as a loading-state fallback while the user record is still being fetched. The change was prompted by community pushback against Trac #65083, which proposed removing the `Howdy` greeting from the admin bar; the new dashboard is treated as the more fitting home for a personal welcome.

## Impact

**Plugin & theme developers**
- No action required. No public hooks, filters, or APIs were added, changed, or removed.

**Site owners / end users**
- The new Gutenberg dashboard landing page will greet logged-in users by name instead of showing a generic `Dashboard` heading.

**Scope**
- Affects only the new Gutenberg dashboard route (`routes/dashboard/stage.tsx`). The classic admin dashboard (`/wp-admin/index.php`), the admin bar, and all other routes are unaffected.
- No breaking changes, no deprecations.

## Technical details

All changes are confined to `routes/dashboard/stage.tsx`.

A `useSelect` call retrieves the current user record via `getCurrentUser()` from `@wordpress/core-data` and derives `greetingName` with the following fallback chain:

```ts
const greetingName =
  currentUser?.name?.trim() ||
  currentUser?.username ||
  currentUser?.slug;
```

Page title assignment (typed as `string` to satisfy TypeScript across all branches):

```ts
// Before
const pageTitle = isCustomizing
  ? __( 'Customize Dashboard' )
  : __( 'Dashboard' );

// After
let pageTitle: string;
if ( isCustomizing ) {
  pageTitle = __( 'Customize Dashboard' );
} else if ( greetingName ) {
  pageTitle = sprintf( __( 'Howdy, %s' ), greetingName );
} else {
  pageTitle = __( 'Dashboard' ); // loading fallback
}
```

Bundle size delta is 0 B — the `getCurrentUser` selector and `sprintf` are already pulled in by the dashboard bundle.

## Contribution

Opened and merged by @jameskoster (commit `1a04ac9`). The PR description frames it explicitly as a response to the community sentiment around Trac #65083. Discussion on the PR itself was limited to automated bot comments (bundle size check, contributor attribution). Co-authored-by credits in the merge commit list @simison and @retrofox alongside @jameskoster.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
