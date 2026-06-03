# Accessibility Improvements in WordPress 7.0

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Joe Dolson
- **Published:** 2026-05-23
- **Tags:** `Updates`, `7.0`, `accessibility`, `dev-notes`, `dev-notes-7-0`
- **Link:** [https://make.wordpress.org/core/2026/05/23/accessibility-improvements-in-wordpress-7-0/](https://make.wordpress.org/core/2026/05/23/accessibility-improvements-in-wordpress-7-0/)

## Summary

WordPress 7.0 delivers a comprehensive accessibility pass across Core, Admin, and the Gutenberg editor, focusing on semantic markup corrections, assistive technology compatibility, and WCAG 2.2 AA compliance. The update enables automatic alt text population from IPTC image metadata, replaces non-semantic link elements with proper buttons in media workflows, and fixes focus management across modal and list table interfaces. Editor components gain explicit keyboard navigation support, forced-colors mode handling, and exported accessibility shortcut utilities. These changes reduce screen reader noise and improve voice control reliability without introducing breaking changes.

## Impact

- **Plugin & Theme Developers:** Author link templates no longer render `title` attributes by default. Media upload workflows now auto-populate alt text from embedded IPTC metadata. If you override admin media modals or theme author outputs, verify that custom markup aligns with the new semantic button structure and focus behaviors.
- **Editor/Gutenberg Plugin Developers:** `ariaKeyShortcut` and `shortcutFormats` are now exported in editor utilities. DataViews components enforce grid keyboard navigation and permanently visible checkboxes. Block templates can now use the `ariaLabel` support flag to skip default serialization when native HTML semantics already provide labeling.
- **Site Owners & Admins:** Admin color contrast errors are resolved, zero-comment notification contrast is improved, and password reset flows pre-populate usernames. Voice control compatibility is extended across the Add Media panel. No configuration or migration steps required.

## Technical details

- **Core PHP & Media:** IPTC photo metadata is now parsed and automatically mapped to the `alt` field during import (`#55535`, `#63895`). The featured image button was refactored from an `<a>` element to a semantic control, with missing ARIA attributes added to meet programmatic identification standards (`#63980`). Media modal tabpanels now correctly receive focus on open (`#63984`).
- **Admin UI:** List tables reduced redundant `Edit` links for screen reader users (`#33002`). Screen reader-only CSS classes now explicitly set `word-break` to prevent horizontal scrolling issues (`#64375`). Post search close buttons gain `cursor: pointer`, and automated contrast thresholds were satisfied for dashboard and zero-comment notifications (`#64313`, `#64811`).
- **Editor & Block API:** Range controls now respect `forced-colors` mode, and fit-text warnings enforce a 12px minimum font size. The DataViews table checkbox is permanently visible, and grid keyboard navigation is implemented. Block template skip links now leverage the HTML API directly. New interface components (Icons block, Gallery lightbox, Connectors screen) ship with validated ARIA roles and focus management.

## Contribution

The 7.0 accessibility improvements were coordinated across multiple Trac tickets spanning Core PHP, admin styling, and the Gutenberg monorepo, driven by WordPress’s stated goal of meeting WCAG 2.2 AA standards for all new/updated code. Key contributors reviewed and merged the patches as part of a consolidated sprint, with editor components specifically reviewed by @sabernhardt and @amykamala. The approach prioritized incremental markup rectification (ARIA attributes, focus management) and semantic API additions (`ariaLabel` serialization handling), aligning with core’s policy of shipping accessibility-compliant interfaces without disrupting existing plugin or theme behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
