# #77162: Core Data: Fix return types on generated save/delete entity actions

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ObliviousHarmony
- **Labels:** `[Type] Bug`, `[Package] Core data`
- **Merged:** [`03396b9`](https://github.com/WordPress/gutenberg/commit/03396b93be3beeaa689ad3d9139ac76903a9c1d1)
- **Discussion:** [#77162](https://github.com/WordPress/gutenberg/pull/77162) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The dynamically generated per-entity action creators in `@wordpress/core-data` (`saveUser`, `deletePost`, `saveTerm`, etc.) were typed as `Promise<void>`, although at runtime they delegate to `saveEntityRecord` / `deleteEntityRecord` and resolve with the entity. This PR corrects the types: save actions now return `Promise<Entity | undefined>` and delete actions `Promise<Entity | false | undefined>`. It is a types-only change with no runtime behavior difference.

## Impact

**Plugin & theme developers (TypeScript)**
- Code that did `(await dispatch( coreStore ).saveFoo( x )) as unknown as Foo` can drop the cast.
- The new return types expose the silent-failure branches. Save can resolve `undefined`; delete can resolve `false` or `undefined`. Code that previously treated the result as `void` may now surface type errors if it uses the value, and consumers should either handle those cases or pass `{ throwOnError: true }`.
- Affects 19 save and 19 delete actions: `Comment`, `GlobalStyles`, `Media`, `Menu`, `MenuItem`, `MenuLocation`, `Plugin`, `PostType`, `Revision`, `Sidebar`, `Site`, `Status`, `Taxonomy`, `Term`, `Theme`, `UnstableBase`, `User`, `Widget`, `WidgetType`.

**Site owners / hosting / REST consumers / JS-only developers**
- No action required. No runtime change.

## Technical details

The change is in `packages/core-data/src/dynamic-entities.ts`. The `SaveActions` and `DeleteActions` mapped types now reuse the `Key extends \`save${infer E}\`` / `\`delete${infer E}\`` extraction already used by `SingularGetters` in the same file, and apply it in the return position. It stays a mapped type rather than hand-written entries, so it tracks `WPEntityTypes` automatically.

The new types reflect the runtime behavior of `packages/core-data/src/actions.js`, as described in the PR:
- `saveEntityRecord` returns `updatedRecord` on success and `undefined` on swallowed errors (visible via `getLastEntitySaveError` unless `throwOnError: true`).
- `deleteEntityRecord` returns `deletedRecord`, initialized to `false` and overwritten with the REST DELETE response on success. It returns `undefined` when no entity config is found.

```ts
// before
saveUser(...): Promise<void>
deleteMedia(...): Promise<void>

// after
saveUser(...): Promise<User | undefined>
deleteMedia(...): Promise<Media | false | undefined>
```

## Contribution

The PR author disclosed that it was developed with AI assistance (Claude Code) and that the runtime behavior was verified by reading `actions.js`. Reviewers listed by the props bot are @ramonjd and @manzoorwanijk. The discussion excerpt provided contains no recorded design debate or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
