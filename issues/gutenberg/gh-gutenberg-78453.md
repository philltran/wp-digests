# #78453: Media Fields: Fix filename truncation with Tooltip

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Feature] Media`
- **Merged:** [`427c83d`](https://github.com/WordPress/gutenberg/commit/427c83d783ae6768dd408d953a9944a97de7ac16)
- **Discussion:** [#78453](https://github.com/WordPress/gutenberg/pull/78453) · 4 comments · 1 reactions

## Summary

Fixes an accessibility issue in the Media Upload Modal where the filename field's Tooltip introduced an unexpected tab stop, disrupting keyboard navigation within DataViews-based lists. The update replaces the JS-based truncation component with CSS truncation and ensures the Tooltip element is removed from the natural tab order.

## Impact

['**Developers:** No code changes or migration steps required. Upgrading automatically resolves the tabindex regression in the Media Library modal.', "**Keyboard Users:** Resolves a behavior where hovering over filenames in the media library's DataViews picker added an unnecessary element to the keyboard focus sequence, making it harder to navigate large lists."]

## Technical details

['The update modifies the **Media Fields** package (`packages/media-fields`) to remove usage of `__experimentalTruncate`.', 'Filename rendering now uses a standard `span` with CSS-based truncation. This approach keeps the full filename in the DOM for screen readers while visually truncating it using styles added to `packages/media-fields/src/filename/style.scss`.', 'To prevent the tooltip from acting as an extra tab stop, the component explicitly sets `tabIndex={ -1 }` on the Tooltip wrapper.']

## Contribution

Merged by @andrewserong following review feedback from @talldan regarding unexpected tab stops. The approach addressed concerns raised during the related PR #78423 to ensure screen readers could still access the full filename without compromising keyboard traversal.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
