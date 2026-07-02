# #75757: Add command palette trigger button to admin bar

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `Backported to WP Core`, `No Core Sync Required`, `[Feature] Command Palette`
- **Merged:** [`432e9f7`](https://github.com/WordPress/gutenberg/commit/432e9f7fc3b950f7e54a58987494980be9ee8995)
- **Discussion:** [#75757](https://github.com/WordPress/gutenberg/pull/75757) · 8 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Adds a persistent clickable trigger button to the WordPress admin toolbar to launch the Command Palette (previously invoked primarily via global keyboard shortcuts). This enhancement improves feature discoverability and provides a direct UI entry point for administrators and editors navigating back-end workflows.

## Impact

- **Site administrators & editors**: Gain a visible, always-available admin bar button to open the Command Palette, reducing reliance on remembering shortcut keys like `Ctrl+K`/`Cmd+K`.
- **Plugin & theme developers**: No public API changes, hooks, or REST schema modifications; this is an internal UI component addition. Developers customizing the admin bar layout or hiding specific menu items should account for the new button's presence in responsive breakpoints.
- **No action required** for typical plugin/theme functionality, but administrators using heavily modified admin bars may need to verify CSS/JS compatibility with the added element.

## Technical details

The merged change injects a clickable trigger element into the default WordPress admin toolbar to activate the Command Palette UI. The diff modifies the `build/scripts/core-commands/index.min.js` bundle, which saw a net increase of 129 B. Review feedback favored an inline script registration pattern for event handling to ensure immediate availability without deferring to later-bound queues, while also noting the importance of a compact visual footprint to prevent admin bar overflow and compliance with core's standard against `title` attributes. The final implementation delivers the functional button trigger as architected in the discussion.

## Contribution

Initiated by @t-hamano to address Trac #64672. Design and architecture feedback was provided by @JosVelasco (advocating for a compact icon to prevent overflow and avoid PHP UA-sniffing), @sabernhardt (noting front-end search integration patterns and core's removal of `title` attributes), and @youknowriad (preferring inline script execution). The patch was co-authored by @brandonpayton, @sabernhardt, @mukeshpanchal27, @westonruter, @ellatrix, and @JosVelasco before being merged and backported to WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
