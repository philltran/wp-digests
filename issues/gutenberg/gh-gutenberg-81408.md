# #81408: RTC: Add fallback identities when user profiles are unavailable

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ingeniumed
- **Labels:** `[Package] Core data`, `[Package] Editor`, `[Feature] Real-time Collaboration`, `[Type] Feature`
- **Merged:** [`bee1b17`](https://github.com/WordPress/gutenberg/commit/bee1b17fb13fd4f9183850dbc26035dbc6435721)
- **Discussion:** [#81408](https://github.com/WordPress/gutenberg/pull/81408) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Gutenberg's real-time collaboration awareness layer previously assumed `resolveSelect( coreStore ).getCurrentUser()` would always resolve — on sites that remove the `/wp/v2/users` REST routes the request fails, `collaboratorInfo` is never published to the Yjs awareness state, and presence consumers end up reading properties off a missing object. This PR makes `BaseAwarenessState.setCurrentCollaboratorInfo()` tolerate a failed or unusable user response and publish a complete session-scoped fallback identity instead: name `Anonymous User`, `id: null`, slug `anonymous-<clientId>`, and no `avatar_urls` so the existing avatar component renders "AU" initials. Session-specific behavior (cursor color, block highlighting, avatar grouping, notification IDs) keys off the Yjs awareness `clientID` when no WordPress user ID exists. It also adds a runtime type guard, `isCollaboratorInfo()`, so malformed peer-supplied awareness data is excluded from presence and debug output rather than trusted because TypeScript says so.

## Impact

- **Site owners / platform teams:** Sites that intentionally unregister the `/wp/v2/users` routes (while keeping, e.g., Application Password endpoints) can now use collaborative editing — the collaborator appears as `Anonymous User` with a distinct cursor color instead of being missing from presence UI. Authentication and document permissions are unchanged and remain the collaboration provider's and server's responsibility; this only supplies presentation data.
- **Plugin & theme developers consuming the awareness API:** The exported `CollaboratorInfo` type in `@wordpress/core-data` is no longer a `Pick<User<'view'>, …>`. Two shape changes to handle: `id` is now `number | null`, and `avatar_urls` is now **optional** (`Partial<User<'view'>['avatar_urls']>`). Any code doing `collaboratorInfo.avatar_urls['48']` or treating `id` as a truthy number needs guarding.
- **Callers of `generateCollaboratorInfo()`:** Signature changed — it now takes `( currentCollaborator: unknown, clientId: number )`. The second argument is required.
- **Debug-data consumers:** `getDebugData().collaboratorMap` keeps its `{ name, wpUserId }` shape, but `wpUserId` can now be `null`, and entries whose awareness state fails `isCollaboratorInfo()` are omitted entirely.
- **Everyone else:** No action required — this is scoped to the experimental real-time collaboration feature and does not alter behavior when `getCurrentUser()` resolves normally.

## Technical details

In `packages/core-data/src/awareness/base-awareness.ts`, `setCurrentCollaboratorInfo()` wraps the user resolution in a `try`/`catch` and passes the Yjs client ID through:

```ts
// Before
const currentUser = await resolveSelect( coreStore ).getCurrentUser();
const collaboratorInfo = generateCollaboratorInfo( currentUser );
this.setLocalStateField( 'collaboratorInfo', collaboratorInfo );

// After
let currentUser: User< 'view' > | undefined;
try {
	currentUser = await resolveSelect( coreStore ).getCurrentUser();
} catch {
	// Fall through with an undefined user so a fallback identity is generated.
}
this.setLocalStateField(
	'collaboratorInfo',
	generateCollaboratorInfo( currentUser, this.clientID )
);
```

`packages/core-data/src/awareness/utils.ts` gains two functions. `hasValidAvatarUrls()` (internal) rejects non-objects and arrays and requires any present `24`/`48`/`96` key to be a string — malformed sizes are dropped individually. The new exported `isCollaboratorInfo( value: unknown ): value is CollaboratorInfo` validates the full record at the peer boundary: `id` must be `null` or a positive integer, `name` a non-empty trimmed string, `slug` a string, `avatar_urls` (if present) valid, `browserType` a string, and `enteredAt` a finite number. `generateCollaboratorInfo()` was widened to `( currentCollaborator: unknown, clientId: number )` and now builds the identity defensively, emitting `{ browserType, enteredAt, id: null, name: 'Anonymous User', slug: \`anonymous-${clientId}\` }` when the input is `undefined` or an unusable response such as `{ id: 1 }` with no `name`.

`packages/core-data/src/awareness/types.ts` redefines `CollaboratorInfo` as a standalone type rather than deriving it from `User< 'view' >`, with `id: User< 'view' >[ 'id' ] | null` and optional `avatar_urls`. In `post-editor-awareness.ts`, `getDebugData()` now filters `getSeenStates()` through `isCollaboratorInfo()` before mapping to `DebugCollaboratorData`, so incomplete peer states no longer produce entries with `undefined` fields.

Unit tests in `awareness/test/` cover the rejected-promise path, the invalid-user-response path, avatar normalization, the `isCollaboratorInfo` accept/reject matrix, and debug-map filtering. A request-scoped test plugin is included that removes the standard Users REST routes while preserving Application Password routes, gated behind a `gutenberg_test_hide_user_rest_routes` cookie for e2e and manual testing.

## Contribution

Authored by @ingeniumed with OpenAI Codex assistance; the props-bot credits @ingeniumed and @chriszarate. The PR record carries no design debate — the only human comment is @ingeniumed noting that the failing `dataviews-list-layout-keyboard.spec.js` e2e failure was a pre-existing flake that re-runs would not clear, and that a separate permanent fix was opened as PR #81739 to unblock the merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
