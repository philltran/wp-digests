# #79540: Tabs: Fix dirty editor state on mount caused by tab-list sync

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Tabs`
- **Merged:** [`51ccfe5`](https://github.com/WordPress/gutenberg/commit/51ccfe58a64a6c8403eaf8b8a48172f4c87cc8b6)
- **Discussion:** [#79540](https://github.com/WordPress/gutenberg/pull/79540) · 10 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a React StrictMode double-mount issue in the Tabs block that incorrectly marked posts as dirty on editor reload. The `useTabListItemsSync` effect previously wrote the `tabs` attribute on every mount because an initial `null` ref bypassed its change guard, causing the non-persistent flag to be consumed by other dispatches and resulting in false dirty state markers for developers testing under StrictMode.

## Impact

- **Developers & Testers**: Resolves a false "dirty" post state on reload when React StrictMode is enabled (standard in local development builds). No impact on production client rendering or saved content.
- **Plugin/Theme Authors**: No action required. This is an internal block-library synchronization fix with no API surface changes.

## Technical details

In `packages/block-library/src/tabs/use-tab-list-items-sync.js`, the `useTabListItemsSync` effect replaces the mutable `prevTabsRef` pattern with a store lookup via `getBlockAttributes( tabListClientId )`. The comparison logic shifts from stringifying the expected tabs (`JSON.stringify`) to a deep equality check using `fastDeepEqual`. This guard prevents `updateBlockAttributes` from firing on initial mount when derived panel labels already match stored data, eliminating the race condition that misfires `__unstableMarkNextChangeAsNotPersistent()` under React StrictMode's double-invoke behavior.

## Contribution

Opened and merged by @Mamaduka (co-authored with @tyxla, @mcsf, and @t-hamano) as part of ongoing Tabs block stabilization (#73230). Review feedback highlighted architectural debts around duplicated `label` attributes and in-editor state modeling, but reviewers approved the immediate sync fix to resolve the StrictMode bug before pursuing deeper refactors.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
