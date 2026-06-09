# #78671: Font Library: Fix focus issue when navigating

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Enhancement`, `[Focus] Accessibility (a11y)`, `[Feature] Font Library`
- **Merged:** [`ddfc403`](https://github.com/WordPress/gutenberg/commit/ddfc40378f7cd3b4459a0cbb81935f9a82f1d44f)
- **Discussion:** [#78671](https://github.com/WordPress/gutenberg/pull/78671) · 5 comments · 0 reactions

## Summary

This PR fixes a focus regression in the Font Library where navigating back from a font detail view incorrectly returns keyboard focus to the first item in the list instead of the previously selected font. The navigator component’s built-in focus-return logic fails when the list is fully unmounted during navigation, causing its DOM reference to become stale. The fix captures the active font’s slug before selection is cleared and programmatically refocuses the corresponding button after the list remounts.

## Impact

- **Editor Users & Accessibility Evaluators**: Resolves a focus displacement issue when using Back navigation in the Fonts panel (Appearance > Fonts), ensuring keyboard and screen reader users can maintain context.
- **Theme/Plugin Developers**: No code changes or configuration updates required; existing Font Library flows automatically inherit the corrected focus behavior.
- **Platform/Hosting Teams**: No migration or compatibility changes needed; this is a client-side component fix with no backend or database impact.

## Technical details

- The implementation modifies the font list's selection handler to persist the currently active font slug in a ref before clearing the selection state on Back navigation.
- Upon remount, the component queries the DOM for the button matching `data-font-slug="<slug>"` and explicitly calls `.focus()` on it, overriding the stale reference that previously triggered fallback to the first focusable element.
- This approach circumvents the navigator's automatic focus-return logic by taking ownership of focus placement after the unmount/remount cycle completes.

## Contribution

Opened and merged by @juanfra (commit ddfc403) under the Font Library feature set with a Focus (a11y) enhancement label. The PR discussion was minimal, consisting primarily of automated size-change reporting (+111 B across edit-site and editor bundles), with no alternative approaches or design debates documented prior to merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
