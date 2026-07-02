# #78387: useCopyToClipboard: Always call onSuccess callback

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Package] Compose`, `Backport to WP Minor Release`
- **Merged:** [`b1969c9`](https://github.com/WordPress/gutenberg/commit/b1969c9545b5e8a08c488e21917fdedef76899fe)
- **Discussion:** [#78387](https://github.com/WordPress/gutenberg/pull/78387) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a regression in the `useCopyToClipboard` hook (`@wordpress/compose`) where the `onSuccess` callback was silently skipped when the trigger element unmounted before the asynchronous clipboard operation resolved. The fix relaxes the mount-state guard to ensure `onSuccess` always fires, restoring expected UX feedback patterns for dynamic components like closing dropdowns or modals.

## Impact

- **Block & Plugin Developers**: Developers using `useCopyToClipboard` can now rely on `onSuccess` triggering reliably regardless of trigger element lifecycle changes (e.g., when a click handler closes the parent menu immediately).
- **No breaking changes**: Callback signatures and return values remain unchanged; consumers receive transparent behavior correction.
- **Action required**: None for existing implementations, though developers relying on this callback for UI state updates should verify behavior in rapidly unmounting components.

## Technical details

Modifies the internal execution flow of `@wordpress/compose`'s `useCopyToClipboard` hook. Previous logic applied an early-return check on an `isActive` flag to prevent side effects on unmounted nodes, which inadvertently suppressed `onSuccess` entirely when the trigger was removed mid-promise. The updated diff removes the unconditional early return for the success callback, allowing it to execute asynchronously while still preserving focus restoration logic tied to the trigger element's state. This ensures the callback fires reliably even when the DOM node is removed before `navigator.clipboard.writeText()` resolves.

## Contribution

Opened and merged by @t-hamano as a direct follow-up to #75723. The PR was flagged for backport to a WordPress minor release to address a 7.0 regression. Co-authored by @mamaduka and @mciampini during review. The implementation focuses strictly on relaxing the `isActive` guard without altering focus restoration behavior, resolving the unmount race condition efficiently.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
