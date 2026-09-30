# #81743: Tabs: make inactive panel content findable in page

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @sanketio
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `First-time Contributor`, `[Block] Tabs`
- **Merged:** [`958580c`](https://github.com/WordPress/gutenberg/commit/958580c1b156ea55bccc6dcb05a87409d231d921)
- **Discussion:** [#81743](https://github.com/WordPress/gutenberg/pull/81743) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tabs block now hides inactive `core/tab-panel` elements with `hidden="until-found"` instead of a bare `hidden`, so browser find-in-page can locate text in unselected tabs and reveal it. When the browser reveals a panel, a `beforematch` handler activates the matching tab. This mirrors the earlier Accordion fix. It also fixes server output: previously every panel was rendered `hidden`, so visitors without JavaScript saw no tab content.

## Impact

- **Site owners / visitors:** Text inside non-selected tabs is now findable with Cmd/Ctrl+F, and the matching tab is selected on reveal. Visitors whose browser does not run the block's script now get the content of every panel instead of none.
- **Plugin & theme developers:**
  - Theme CSS that relied on `.wp-block-tab-panel[hidden] { display: none }` from core no longer gets it. Core's stylesheet no longer forces `display: none` on `[hidden]`; the UA rule still hides a bare `hidden` panel.
  - A panel hidden with `until-found` keeps a layout box, so custom margin, padding or height styles on panels may show as extra space while hidden. Core now zeroes `margin-block-start` and `padding` on `[hidden]` panels.
  - Anything reading or asserting the old `!state.isActiveTab` binding or the saved `tabindex="0"` on panels will see different values. Panels now get `tabindex` bound to `state.tabIndexAttribute`.
- **Browsers:** Browsers without `until-found` support fall back to ordinary hidden behavior; the PR notes this was the reason not to resolve `hidden` on the server.
- No migration is required, and no block markup is invalidated (saved content is unchanged).

## Technical details

**`tab-panel/index.php` (`block_core_tab_panel_render`)** now sets three directives on the panel via the tag processor:

```php
// before
$tag_processor->set_attribute( 'data-wp-bind--hidden', '!state.isActiveTab' );

// after
$tag_processor->set_attribute( 'data-wp-bind--hidden', 'state.isHidden' );
$tag_processor->set_attribute( 'data-wp-bind--tabindex', 'state.tabIndexAttribute' );
$tag_processor->set_attribute( 'data-wp-on--beforematch', 'actions.handleBeforeMatch' );
```

Per the PR, the old expression was evaluated server-side where `state.isActiveTab` is undefined, so `!undefined` was `true` and all panels were emitted `hidden`. A getter the server cannot resolve leaves panels unhidden until hydration, as `core/accordion` does.

**`tabs/view.js`:**
- New getter `state.isHidden` returns `null` for the active tab and `'until-found'` otherwise.
- New action `actions.handleBeforeMatch` reads `state.tabIndex` and calls `actions.setActiveTab( tabIndex )` when it is not `null`.
- The existing `state.tabIndexAttribute` (0/-1 against the active tab) is now shared by tab buttons and panels, so hidden panels leave the tab sequence.

**`tab-panel/style.scss`:** `&[hidden]` is removed from the `display: none !important` rule (`&:empty` remains). A new `&[hidden]` rule sets `margin-block-start: 0` and `padding: 0 !important`, because `until-found` applies `content-visibility: hidden`, which skips contents but not the element's own box.

**Tests:** a PHPUnit test (`test_should_leave_hiding_tab_panels_to_the_client`) asserts no server-side `hidden` and the three directives. A new e2e `Frontend functionality` block checks `hidden="until-found"`, zero height, `tabindex="-1"`, and that dispatching `beforematch` selects the tab. A `CHANGELOG.md` entry was added.

## Contribution

Opened by first-time contributor @sanketio to close #81712, following the earlier Accordion fix (#73443 / #74744). @t-hamano steered the design away from resolving `hidden` on the server, which would have left inactive panels unreachable in browsers lacking `until-found` support, and also asked the author to keep comments concise. @hbhalodia tested it locally and asked for a merge because it blocked #81744. The PR states it was authored with Claude Code.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
