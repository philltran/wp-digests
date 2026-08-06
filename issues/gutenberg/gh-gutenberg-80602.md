# #80602: URLInput: Skip search requests while an IME composition is in progress

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Bug`, `[Package] Block editor`
- **Merged:** [`61e4398`](https://github.com/WordPress/gutenberg/commit/61e43986ac4496c1c4db33475ff02739755f361b)
- **Discussion:** [#80602](https://github.com/WordPress/gutenberg/pull/80602) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `URLInput` component in the Block Editor now suppresses debounced `/wp/v2/search` requests while an Input Method Editor (IME) composition is active. Previously, every intermediate keystroke during CJK or Japanese typing triggered a search request; the fix tracks `compositionstart` and `compositionend` events, cancels pending updates during composition, and fires a single request with the confirmed value once composition ends. This eliminates redundant network traffic and aligns the Link UI with existing rich text composition handling.

## Impact

- **Block editor users**: Smoother typing experience in the Link UI when using IME keyboards; no more spammy search requests during composition.
- **Plugin & theme developers**: No breaking changes or API modifications. The `URLInput` component's public props and behavior remain identical; only internal request timing changes.
- **Hosting & platform teams**: Reduced load from unnecessary `/wp/v2/search` calls during IME usage.
- **No action required** for existing codebases or custom integrations.

## Technical details

In `packages/block-editor/src/components/url-input/index.js`, a new `isComposing` state tracks IME activity. `handleCompositionStart` sets the state to `true` and calls `debouncedUpdateSuggestions.cancel()` to purge any queued updates. `handleCompositionEnd` resets the state to `false`. The `useEffect` driving `debouncedUpdateSuggestions` now includes `isComposing` in its dependency array, ensuring the effect re-runs once composition ends. The input element receives `onCompositionStart` and `onCompositionEnd` handlers. Jest tests verify that intermediate composition values do not trigger `fetchLinkSuggestions`, while the confirmed value triggers exactly one request. An e2e test uses Playwright's CDP session (`Input.imeSetComposition`) to simulate IME composition in Chromium, verifying that no `/wp/v2/search` requests fire until `Input.insertText` confirms the composition.

## Contribution

Opened by @jorgefilipecosta and merged with co-authors @Mamaduka and @fumikito, the change implements a straightforward state-tracking patch to align `URLInput` with existing rich text composition handling. The team opted for a CDP-driven e2e test to simulate IME composition in Chromium, bypassing Playwright's native IME limitations, rather than relying on manual testing or third-party IME plugins. No alternative architectural approaches were debated in the review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
