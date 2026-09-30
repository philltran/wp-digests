# #81813: Fix: Post Editor: 403 on /wp/v2/settings at boot for users without manage_options cap

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @hbhalodia
- **Labels:** `[Type] Bug`, `[Type] Regression`, `[Package] Edit Post`, `Backported to WP Core`
- **Merged:** [`0aa2071`](https://github.com/WordPress/gutenberg/commit/0aa2071fecab991fae2539cc7dabd217c5240141)
- **Discussion:** [#81813](https://github.com/WordPress/gutenberg/pull/81813) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The post editor's boot-time preloading (`preloadResolutions()` in `packages/edit-post/src/index.js`) no longer requests `/wp/v2/settings` unconditionally. Users without the `manage_options` capability, such as Authors, Editors, and Contributors, previously got a 403 and a console error every time they opened Posts → Add New. The fix gates the site-entity fetch behind a `canUser( 'read', site )` check and reworks how the global styles record is preloaded so it uses the correct request context for low-capability users.

## Impact

- **Site owners / editors without `manage_options`:** The 403 on `/wp/v2/settings` and the resulting console error at editor boot no longer occur. The PR discussion also mentions minor visual differences attributed to the failed request. They are described as non-blocking, and the source does not detail them.
- **Plugin and theme developers:** No API, hook, or filter changes. If you have tests or monitoring that assert on console errors or failed network requests when loading the editor as a low-privilege role, those should now pass cleanly.
- **Hosting / platform teams:** Fewer 403 responses in logs from the editor for non-admin roles.
- **Version note:** The author states the bug has been present since Gutenberg 23.3. Maintainers agreed to backport to the 7.1 minor release (the PR carries the `Backported to WP Core` label).
- **Action required:** None.

## Technical details

The change is confined to `preloadResolutions()` in `packages/edit-post/src/index.js`, which preloads entity records and capability checks in phases.

**Phase 1** (initial parallel batch): `core.getEntityRecord( 'root', 'site' )` is removed. The `core.canUser( 'read', { kind: 'root', name: 'site' } )` check stays, so the capability is resolved here.

**Phase 2**, where derived data is read from state:
- `core.getEntityRecord( 'root', 'site' )` is pushed onto `tasks` only if `coreSelect.canUser( 'read', { kind: 'root', name: 'site' } )` is truthy.
- For the global styles ID, the preloaded capability check changes from `canUser( 'read', ... globalStyles )` to `canUser( 'update', ... globalStyles )`. The direct `getEntityRecord( 'root', 'globalStyles', id )` call is removed from this phase.

**Phase 3** (new), for requests whose capability only resolved in phase 2:

```js
if ( globalStylesId ) {
	if ( coreSelect.canUser( 'update', { kind: 'root', name: 'globalStyles', id: globalStylesId } ) ) {
		await core.getEntityRecord( 'root', 'globalStyles', globalStylesId );
	} else {
		// Fetch for non-admin users using view context.
		await core.getEntityRecord( 'root', 'globalStyles', globalStylesId, { context: 'view' } );
	}
}
```

Users who can update global styles get the default (edit-context) record. Everyone else gets it with `{ context: 'view' }`. The surrounding `try/catch` is unchanged, so resolver failures still don't block render. No REST schema, hook, or database changes.

## Contribution

Opened by @hbhalodia to close issue #81812, with Claude Code disclosed as used for issue review, drafting, and implementation, and the author stating the result was reviewed and tested by hand. @Mamaduka is credited by the props bot. @t-hamano asked whether the bug caused real problems beyond console errors. @hbhalodia answered that there were only minor visual differences, not blockers, but that Gutenberg 23.3 onward is affected. On that basis @t-hamano agreed to backport it to the 7.1 minor release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
