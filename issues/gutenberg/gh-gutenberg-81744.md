# #81744: Fix: Tabs block: Anchor links in tab panels are not functional

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @hbhalodia
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Tabs`
- **Merged:** [`67af03c`](https://github.com/WordPress/gutenberg/commit/67af03c5b80d144f1b2af6bd4471032911977973)
- **Discussion:** [#81744](https://github.com/WordPress/gutenberg/pull/81744) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tabs block now activates the correct tab when the URL hash targets an element inside a non-active tab panel, and scrolls to that element. Previously only a hash equal to a panel's own ID was handled, and only on initial load, so anchors set on blocks inside inactive panels (or links clicked after page load) did nothing. The behavior now mirrors the Accordion block's hash handling.

## Impact

- **Site owners / content editors:** Deep links like `example.com/page#anchor` and in-page links to a block with an HTML anchor inside a tab panel now open the right tab and scroll to the target. No content changes needed.
- **Plugin & theme developers:** No public PHP or JS API change. Code that references the Interactivity callback `callbacks.onTabsInit` (e.g. custom markup or overrides of the rendered `data-wp-init`) should switch to `callbacks.activateTabByHash`; the old callback name is removed from the store.
- **Nested tabs:** The lookup walks up ancestor panels so nested Tabs blocks resolve to the panel belonging to the block handling the event.
- No action required for most sites.

## Technical details

**`tabs/index.php`:** `block_core_tabs_render_block_callback()` now sets `data-wp-init` to `callbacks.activateTabByHash` (previously `callbacks.onTabsInit`) and adds `data-wp-on-window--hashchange` with the same callback, so it runs both on init and on every `hashchange`.

**`tabs/view.js`:** `callbacks.onTabsInit` is replaced by `callbacks.activateTabByHash`. The old version compared `window.location.hash` to entries in `state.tabsList` (panel IDs), so it only matched a hash that was a panel's own ID. The new logic:

1. Returns early if `tabsList` is empty or `document.querySelector( ':target' )` finds nothing.
2. If `targetElement.id` is in `tabsList`, calls `actions.setActiveTab( index, true )` and returns.
3. Otherwise, starting from `targetElement.closest( '.wp-block-tab-panel' )`, walks up through ancestor panels until one whose `id` is in `tabsList` is found (handles nested tabs).
4. Calls `actions.setActiveTab( tabIndex )`, then `window.setTimeout( () => targetElement.scrollIntoView(), 0 )` so scrolling occurs after the panel becomes visible.

The diff also adds a CHANGELOG entry and two E2E tests (load with `#target` hash, and clicking an in-page link to an anchor inside an inactive panel). The existing find-in-page test's `createNewPost()` is moved to a `beforeEach`.

## Contribution

The fix ports the approach from the Accordion block's hash handling (#73357), and the author disclosed using Claude Code for the implementation and tests. In review, @t-hamano found several issues in the Accordion-derived implementation and noted the Accordion may need similar fixes; @hbhalodia agreed to open a follow-up for that. Failing E2E tests were shown to also fail on trunk. One open review question was left tied to a separate browser find-in-page PR (#81743).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
