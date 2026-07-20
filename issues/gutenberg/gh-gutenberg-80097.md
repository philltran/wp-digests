# #80097: Tabs: Wrap tab list onto multiple lines by default

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mustafabharmal
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Tabs`
- **Merged:** [`8ea6d6a`](https://github.com/WordPress/gutenberg/commit/8ea6d6a708cae64c471463f65aa76d96c6802904)
- **Discussion:** [#80097](https://github.com/WordPress/gutenberg/pull/80097) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `core/tab-list` block now wraps its tab items onto multiple lines by default when horizontal space is insufficient, replacing the previous single-line behavior. This change eliminates horizontal overflow in the block editor and on the front end, providing a more responsive out-of-the-box experience without requiring manual layout configuration.

## Impact

- **Block & theme developers:** The default layout for `core/tab-list` now uses `flexWrap: "wrap"`. Existing implementations that relied on the previous single-line behavior may see tabs break into multiple rows on narrower viewports. No code changes are required to adopt the new behavior, but visual regression testing is recommended if custom CSS overrides flex layout.
- **Site owners & editors:** Tabs will automatically wrap instead of causing horizontal scroll or clipping in the block editor and on the front end. No configuration changes needed.
- **Platform & hosting teams:** No action required. This is a client-side block rendering adjustment with no server-side or database impact.

## Technical details

The diff updates `packages/block-library/src/tab-list/block.json`, changing the `layout.default.flexWrap` value from `"nowrap"` to `"wrap"`. This single configuration shift alters the block's default flex container behavior:
```json
// Before
"flexWrap": "nowrap"
// After
"flexWrap": "wrap"
```
The block continues to render via the standard block supports layout system, but now automatically breaks tab items into multiple rows when the container width is exceeded. No new hooks, filters, or block.json fields are introduced, and the change is applied globally to all `core/tab-list` instances.

## Contribution

Opened by @Mustafabharmal to address horizontal overflow in the Tabs block (#80096). Reviewers (@hanneslsm, @jasmussen, @t-hamano) discussed adding scroll/overflow controls but agreed to ship simple multi-line wrapping as the default to stabilize the block. The implementation was simplified to a single `block.json` configuration change without exposing UI controls, and merged with co-authors @t-hamano, @hanneslsm, @jasmussen, and @himanshupathak95.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
