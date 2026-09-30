# #82821: Data: Defer async-mode useSelect notification to idle time

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Data`, `[Package] Editor`
- **Merged:** [`8dc9a20`](https://github.com/WordPress/gutenberg/commit/8dc9a208da8c637d549b53a5398c50b591530610)
- **Discussion:** [#82821](https://github.com/WordPress/gutenberg/pull/82821) · 14 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Async-mode `useSelect` subscribers in `@wordpress/data` no longer each register a separate listener on the store. All async instances in a given registry now share a single `registry.subscribe` call per store; that one subscription's only synchronous work is scheduling an idle-time flush via `renderQueue.add`, and the flush then queues each individual hook instance into the existing render queue. In the post editor with a 953-block post, a single keystroke previously invoked 5,016 `core/block-editor` listeners; it now invokes 693, eliminating roughly 4.6 ms of listener sweep from the input task.

## Impact

- **Site owners / editors:** No visible behavior change. Typing and selection in the block editor are measurably faster on posts with many blocks (the `type` perf metric improved ~3–8% in the Gutenberg perf suite). No action required.
- **Plugin & theme developers:** No API change. `useSelect` in async mode (the default under `AsyncModeProvider`) works identically from the consumer's perspective. No code, configuration, or migration changes needed.
- **Hosting & platform:** Transparent performance gain for front-end JS bundles; `build/scripts/data/index.min.js` grew by 201 B and `build/scripts/editor/index.min.js` by 73 B.
- **Headless & REST consumers:** Unaffected; this is a client-side data-layer optimization only.

## Technical details

The core change is in `packages/data/src/components/use-select/index.ts`.

**New `subscribeDeferred` helper.** A module-level `WeakMap<DataRegistry, Map<string, AddToBucket>>` (`deferredBuckets`) caches one shared subscription per (registry, storeName) pair. The first async listener for a store creates a single `registry.subscribe(() => renderQueue.add(listeners, flush), storeName)` call. The `flush` iterates a `Set<DeferredListener>` and calls `renderQueue.add(context, callback)` for each entry, preserving per-hook-instance coalescing and time-slicing. When the last listener is removed, the bucket is deleted, the pending `renderQueue` entry is cancelled, and the store subscription is torn down.

**`listenToStore` branch.** The `Store` component's `subscribe` function now routes through a `listenToStore(storeName)` helper: async mode calls `subscribeDeferred`, sync mode calls `registry.subscribe(onStoreChange, storeName)` directly. The per-store unsubscribe handles are stored in a `Map<string, VoidFunction | undefined>` (`unsubs`) instead of a flat array.

**Mode-switch handling.** A new `switchMode()` method on the subscriber object iterates all active subscriptions and calls `resubscribeStores()`, which unsubscribes each store and re-subscribes via `listenToStore` (picking the correct tier for the new mode). This runs in place—without handing React a new `subscribe` identity—so a dispatch cannot slip through an unsubscribed gap. The `Store` render body records `wasAsync` before updating `lastIsAsync` and calls `subscriber.switchMode()` when the mode flips.

**Editor-side companion fix.** `packages/editor/src/components/provider/disable-non-page-content-blocks.js` adds an `appliedModesRef` (`useRef(new Map())`) so the `useEffect` only dispatches `setBlockEditingMode` / `setBlockDisabled` for blocks whose mode actually changed, avoiding a set/unset cycle that would briefly flip `inert` on a wrapper and blur the focused element.

**Tests.** Nine new jsdom tests in `packages/data/src/components/use-select/test/index.jsdom.test.jsx` cover: shared subscription count (3 async hooks → 1 `registry.subscribe` call), multi-store coalescing (two store updates → one recomputation), sync→async and async→sync mode switches, a dispatch from a `useLayoutEffect` during a mode switch, a dispatch from a sibling `useEffect`, a dispatch during a flush (scheduling a second flush), subscription recreation after the last subscriber unmounts, and late store registration not sharing the registry-wide fallback.

## Contribution

Opened by @Mamaduka, who credited @ellatrix's PR #82815 as the starting idea and noted the draft was produced with Claude assistance. @Mamaduka initially flagged the PR as a surface-level share and said they would resume work the following week. @jsnajdr raised a pointed question in review: if 500 individual `renderQueue.add` calls are being replaced by a two-level map lookup, why is the new path faster? @youknowriad noted the perf numbers looked promising but asked about e2e test failures. @Mamaduka traced the failures to commit `1ce3b32`, an issue CodeRabbit had flagged (a stale-data window when a subscription is being replaced during a mode switch). The PR was merged at `8dc9a20` after those fixes landed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
