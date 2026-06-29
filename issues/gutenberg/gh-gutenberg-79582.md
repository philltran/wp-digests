# #79582: Tabs: Fix rich text label comparison when syncing the list

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Tabs`
- **Merged:** [`3b0a832`](https://github.com/WordPress/gutenberg/commit/3b0a8329aab0ac89003f88d6b4dd4385848a2ecc)
- **Discussion:** [#79582](https://github.com/WordPress/gutenberg/pull/79582) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a runtime state-sync bug in the core Tabs block where rich text label comparisons incorrectly marked saved posts as dirty after editor reloads. The change replaces a deep-equality check with a serialized JSON comparison, ensuring structurally identical label arrays are recognized as equal without triggering unnecessary re-renders or save triggers.

## Impact

- **Plugin & Theme Developers:** No action required. This is an internal fix to the `core/block-library` package.
- **Editorial Teams & Site Owners:** Prevents false-positive "dirty state" prompts on reload, eliminating accidental unsaved-work warnings and extra database writes during routine editing workflows.
- **Hosting & Platform Teams:** Negligible impact. The change modifies a React sync hook without altering public APIs, block schemas, or backend logic.

## Technical details

The fix modifies `packages/block-library/src/tabs/use-tab-list-items-sync.js` within the `useTabListItemsSync` custom hook. Following PR #79554, merged labels become `RichTextData` objects at runtime, which `fastDeepEqual` fails to match correctly even when their string values align. The diff strips the external `fast-deep-equal` import and swaps the equality check for a native JSON serialization comparison:
```js
// Before
if ( fastDeepEqual( newTabs, currentTabs ) ) {
    return;
}

// After
if ( JSON.stringify( newTabs ) === JSON.stringify( currentTabs ) ) {
    return;
}
```
This aligns the comparison strategy with the runtime shape of the `tabs` attribute array, correctly short-circuiting sync updates when derived and stored values are functionally identical.

## Contribution

Opened and merged by @Mamaduka as a direct follow-up to PR #79554, which initially changed label rendering to use `RichTextData`. Review feedback from @t-hamano identified the runtime comparison mismatch, prompting the switch from `fast-deep-equal` to `JSON.stringify`. The change was landed in commit `3b0a832` with straightforward implementation; no alternative approaches were debated given the narrow scope of the bug.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
