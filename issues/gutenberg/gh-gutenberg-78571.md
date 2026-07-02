# #78571: Custom HTML: Fix scrollbar becoming non-functional after switching tabs

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mustafabharmal
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] HTML`, `Backported to WP Core`
- **Merged:** [`8d67538`](https://github.com/WordPress/gutenberg/commit/8d6753830067140240d9337a631746cf4ee72baf)
- **Discussion:** [#78571](https://github.com/WordPress/gutenberg/pull/78571) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Resolves a regression in the Custom HTML block’s edit modal where scrollbars become non-functional after switching between the HTML, CSS, and JS tabs. Autosize calculations were triggering prematurely while hidden tab panels were still mounting, leaving dimensions incorrect until a subsequent input event occurred.

## Impact

- **Block editors & site admins**: No action required. Fixes a UI glitch in the `Custom HTML` block’s edit modal.
- **Plugin/Theme developers**: No impact on PHP APIs, REST routes, or `block.json` schemas.
- **Developers with `unfiltered_html` capability**: Tab switching in the CSS/JS panels will now scroll correctly without requiring manual input to re-trigger layout calculations.

## Technical details

In `packages/block-library/src/html/modal.js`, the `HTMLEditModal` component now passes the `async` prop to all three `<PlainText>` instances rendering the HTML, CSS, and JS editors within their respective `Tabs.TabPanel` wrappers. This leverages `PlainText`'s internal async autosize logic, deferring the initial size calculation until after React completes the tab visibility updates and DOM measurements are accurate. No other props, state variables, or event handlers were modified.

## Contribution

Reported by @devmasum52 (via Trac #65507 and GitHub #78545), the fix was implemented by @Mustafabharmal and reviewed by @youknowriad, @masteradhoc, and others. It was merged via PR #78571 and cherry-picked to the `wp/7.0` branch for inclusion in WordPress 7.0.1.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
