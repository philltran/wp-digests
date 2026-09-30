# #83067: Data: Stabilize `__unstableNormalizeArgs` as `normalizeArgs`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Data`, `[Package] Core data`
- **Merged:** [`a0550eb`](https://github.com/WordPress/gutenberg/commit/a0550ebb2503d8a6554ad0447ddb1325aca8794d)
- **Discussion:** [#83067](https://github.com/WordPress/gutenberg/pull/83067) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/data` package stabilizes the selector-argument normalization property, renaming `__unstableNormalizeArgs` to `normalizeArgs`. Selectors can now define `normalizeArgs` to coerce their arguments (e.g., converting a string ID to a number) before the selector and resolver execute. The old `__unstableNormalizeArgs` name is still honored as a fallback, so existing stores continue to work without changes. The core-data `getEntityRecord` selector now exposes the method under the stable name.

## Impact

- **Plugin & theme developers (custom stores):** If you defined `__unstableNormalizeArgs` on a selector, rename it to `normalizeArgs`. The old name still works, but the stable name is now the documented, supported API. No breaking change.
- **Plugin & theme developers (new stores):** You can now rely on `normalizeArgs` as a stable, documented property without the `__unstable` caveat.
- **Core-data consumers:** `getEntityRecord` now exposes `normalizeArgs` instead of `__unstableNormalizeArgs`. Code that reads `getEntityRecord.__unstableNormalizeArgs` directly will need to switch to `getEntityRecord.normalizeArgs`.
- **No action required** if you never referenced the normalization property directly or only called selectors through the registry (the data layer handles the fallback internally).

## Technical details

In `packages/data/src/redux-store/index.ts`, a new helper `getNormalizeArgs(selector)` returns `selector.normalizeArgs ?? selector.__unstableNormalizeArgs`. The `normalize()` function and the bound-selector assignment in `createReduxStore` now route through this helper, so both property names are accepted at runtime.

The `SelectorLike` interface retains both properties:

```ts
interface SelectorLike {
  normalizeArgs?: ( args: unknown[] ) => unknown[];
  __unstableNormalizeArgs?: ( args: unknown[] ) => unknown[];
}
```

The public `BoundSelector` type in `packages/data/src/types.ts` and the `GetEntityRecord` interface in `packages/core-data/src/selectors.ts` now declare only `normalizeArgs`:

```ts
// Before
__unstableNormalizeArgs?: ( args: any[] ) => any[];

// After
normalizeArgs?: ( args: any[] ) => any[];
```

In `mapSelectorWithResolver`, the forwarded property is now `selectorResolver.normalizeArgs = selector.normalizeArgs` (the stable name only; the legacy fallback is handled earlier by `getNormalizeArgs` when the bound selector is created).

`getEntityRecord` in `packages/core-data/src/selectors.ts` is updated from `getEntityRecord.__unstableNormalizeArgs = …` to `getEntityRecord.normalizeArgs = …`.

The README example was also corrected to include the `state` parameter in the selector signature and to use a non-mutating spread (`const newArgs = [ ...args ]`) instead of mutating the input array in place.

## Contribution

Opened by @Mamaduka as a direct follow-up to the review discussion on PR #82638 (which introduced the normalization feature alongside `getResolutionArgs`). @jsnajdr is credited as co-author. The PR carried only bot comments (no human review discussion in the thread), and the change was merged as a straightforward rename-with-fallback. The PR notes it was AI-assisted (Claude).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
