# #78454: Media Editor Modal: Fix unexpected tab stop on date fields in the Details sidebar

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Feature] Media`, `[Type] Experimental`
- **Merged:** [`87e0a9b`](https://github.com/WordPress/gutenberg/commit/87e0a9b3439c975012c3a3e60774d61118b184da)
- **Discussion:** [#78454](https://github.com/WordPress/gutenberg/pull/78454) · 6 comments · 0 reactions

## Summary

The experimental Media Editor Modal no longer creates an unexpected tab stop on the "Date added" field within the Details sidebar. Previously, tooltip-based implementations caused the field to be keyboard-focusable despite being non-interactive. This change ensures the field is skipped during keyboard navigation while preserving the full date string for assistive technologies via an aria-label and maintaining hover-triggered tooltips.

## Impact

- **Editor/Block developers**: No action required. The fix applies exclusively to the experimental Media Editor Modal UI flow.
- **Users relying on screen readers or keyboard navigation**: Improved focus management in the media editor's Details sidebar; non-interactive date fields are now correctly excluded from the tab order.
- **Breaking changes / deprecations**: None. This is a purely behavioral improvement within an experimental feature.

## Technical details

The diff modifies the render output for the non-interactive “Date added” field in the Details sidebar component. To prevent the element from participating in the DOM tab order, `tabIndex={ -1 }` is applied directly to the wrapper/container that previously exposed focusability due to its tooltip integration. Concurrently, the full date string is passed to an `aria-label` attribute on the underlying element to ensure screen readers can still access the value without exposing a visible focus ring or interactive control.

**Before/after behavior shift:**
```jsx
// Before: Default tabIndex allowed keyboard nav to land on the tooltip wrapper
<div className="editor-media__details-date">
  {truncateDate(getMediaDate(mediaId))}
</div>

// After: Explicit tabindex control + aria-label for screen readers, preserving hover tooltips
<div
  className="editor-media__details-date"
  tabIndex={ -1 }
  aria-label={ getMediaDate(mediaId) }
>
  {truncateDate(getMediaDate(mediaId))}
</div>
```
No core REST routes, `block.json` schemas, PHP APIs, or database tables are affected. The test suite is extended to validate keyboard tabbing behavior in the Details sidebar and confirm that hover states still trigger correctly.

## Contribution

Opened by @andrewserong as part of a broader accessibility hardening effort for the experimental Media Editor Modal (#73771). The PR was reviewed and merged with commit `87e0a9b` following quick validation from @ramonjd. It was explicitly split out to isolate the date field behavior after a companion fix for the filename field (#78453) addressed the same tooltip-focusability pattern elsewhere in the sidebar. No alternative approaches were debated; the implementation directly resolved the focus order issue through standard ARIA and tabIndex attributes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
