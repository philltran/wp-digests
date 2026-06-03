# #78641: Editor: Fix keyboard activation of the template actions preview

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Editor`
- **Merged:** [`3a453c3`](https://github.com/WordPress/gutenberg/commit/3a453c3567fd1fc824d942d5006e2753b281030c)
- **Discussion:** [#78641](https://github.com/WordPress/gutenberg/pull/78641) · 3 comments · 0 reactions

## Summary

The "Change template" preview thumbnail in the block-theme template actions panel (Inspector sidebar) was using React's deprecated `onKeyPress` event handler, which fires on any character-producing key. As a result, pressing any letter key while the preview `<div role="button" tabIndex={0}>` was focused would incorrectly open the "Choose a template" swap modal. The fix replaces `onKeyPress` with a `onKeyDown` handler gated on `ENTER` and `SPACE` key codes, aligning the element's keyboard behavior with the ARIA button interaction pattern and eliminating the accidental modal trigger.

## Impact

**Plugin & theme developers / site editors using block themes**
- No action required. This is a bug fix to editor UI behavior only — no public APIs, hooks, or block.json fields changed.

**Site editors / keyboard users**
- Pressing any letter key while the template preview is focused no longer opens the "Choose a template" modal. `Enter` and `Space` activate it as expected; `Space` also correctly calls `event.preventDefault()` to suppress page scroll.

## Technical details

The changed component lives in the `@wordpress/editor` package (only `build/scripts/editor/index.min.js` changed, +59 B). The affected element is the template thumbnail `<div role="button" tabIndex={0}>` rendered when `hasSwapTargets` is truthy in the template actions panel.

**Before:**
```jsx
onKeyPress={ () => setIsSwapModalOpen( true ) }
```

**After:**
```jsx
import { ENTER, SPACE } from '@wordpress/keycodes';

onKeyDown={ ( event ) => {
    if ( event.keyCode === ENTER || event.keyCode === SPACE ) {
        event.preventDefault();
        setIsSwapModalOpen( true );
    }
} }
```

The `onClick` handler, `role`, `tabIndex`, `aria-label`, and the `Tooltip.Root` / `Tooltip.Trigger` composition are all unchanged. The pattern mirrors the existing keyboard handler in `packages/block-editor/src/components/default-block-appender/index.js`.

## Contribution

Authored and submitted by @ciampo as a follow-up to #78466, where the `onKeyPress` issue was surfaced in a review thread. @mirka is credited as a co-author/reviewer. The PR carried 3 comments (bot-generated size and props reports) with no substantive design debate; the approach — matching the `default-block-appender` pattern — was identified in the #78466 discussion before this PR was even opened.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
