# #80685: DataForm: Stop card and details validation from hijacking focus

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`2b91675`](https://github.com/WordPress/gutenberg/commit/2b91675404763242f18be40c9a8570021bac4fbd)
- **Discussion:** [#80685](https://github.com/WordPress/gutenberg/pull/80685) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a focus-hijacking bug in the DataForm component where validation errors in card and details layouts would interrupt the natural tab sequence by moving focus to the first invalid field on every blur. The change defers validation until focus leaves the container, reveals errors via a synthetic invalid event that preserves focus, and ensures only trusted submission events trigger native focus movement.

## Impact

- **Plugin & theme developers using DataViews/DataForm:** No code changes required. The tab sequence in card and details layouts now behaves predictably without jumping to invalid fields.
- **Accessibility users:** Improved keyboard navigation and screen reader announcements when validation errors appear without focus displacement.
- **Headless/REST consumers:** Unaffected.
- **Action required:** None. This is a transparent UX improvement that maintains backward compatibility with native form submission.

## Technical details

The core behavior shift lives in `packages/components/src/validated-form-controls/control-with-error.tsx`, which now checks `event.isTrusted` before moving focus to the first invalid control. Trusted events (form submission, `reportValidity()`) retain native focus behavior, while synthetic invalid events only display the error message. In `packages/dataviews`, the card and details layouts replaced the `useReportValidity` hook with `useRevealValidity`, switching from per-field blur listeners to a container-level `__experimentalUseFocusOutside` hook. A new `getValidationMessage` utility counts invalid fields and formats them for `speak()` announcements. Developers can now reveal errors without displacing focus using this pattern:

```tsx
ref.current?.dispatchEvent(
  new Event( 'invalid', { cancelable: true } )
);
```

This replaces the previous internal blur-triggered validation that called `reportValidity()` on every field exit, which forced focus to the first error.

## Contribution

Opened and merged by @jorgefilipecosta, with co-authors including @mirka, @ntsekouras, @ciampo, and @oandregal. The PR addressed issue #76832, where the existing blur-triggered validation broke keyboard navigation. Review focused on ensuring the synthetic event approach didn't regress native form submission behavior, leading to the `event.isTrusted` guard. The team added comprehensive Storybook documentation and unit tests to verify the new focus-preserving pattern, and replaced the internal `useReportValidity` hook with `useRevealValidity` across card, details, and panel layouts.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
