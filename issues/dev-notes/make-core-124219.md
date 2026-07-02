# What’s new in Gutenberg 23.5? (July 1, 2026)

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Bero
- **Published:** 2026-07-01
- **Tags:** `General`, `block-editor`, `core-editor`, `gutenberg`, `gutenberg-new`
- **Link:** [https://make.wordpress.org/core/2026/07/01/whats-new-in-gutenberg-23-5-july-1-2026/](https://make.wordpress.org/core/2026/07/01/whats-new-in-gutenberg-23-5-july-1-2026/)
- **Usefulness:** 4/5

## Summary

Gutenberg 23.5 introduces experimental media editor cropping controls, brings the media modal to the Cover block, and replaces rigid device presets with a freely resizable canvas alongside dynamic breakpoint toggles. It also expands Global Styles with text shadow support, standardizes Icon block defaults, and enables per-post-type disabling of real-time collaboration.

## Impact

- **Plugin & theme developers**: The Gutenberg plugin now requires WordPress 6.9+. `ThemeProvider` is exported directly from `@wordpress/theme`, shifting it from an internal experiment to a supported public API for consuming core theming tokens.
- **Block developers**: The Icon block now inserts `core/info` by default via `block.json` instead of a runtime effect, and exposes flip/rotate controls. The Cover block now leverages the experimental media editor modal for inline image cropping.
- **Editors & site owners**: Device preview is unified; dragging the canvas resizes it continuously while preset toggles snap to breakpoints. Real-time collaboration can be disabled per post type via editor settings.

## Technical details

- **Media Editor & Cover Block**: The experimental media editor gains a magnified crop canvas, pixel-snapping handles, and fixed keyboard resizing for locked aspect ratios. The Cover block now opens this modal for inline cropping operations.
- **Editor Viewport Logic**: Device preview dropdown and resize handles are unified; dragging the canvas sets an exact pixel width independent of Desktop/Tablet/Mobile presets. A toggle in the dropdown enables responsive editing, triggering visibility changes as blocks hit their breakpoints.
- **Global Styles & Icon Block**: `textShadow` is added to the Style Engine schema for programmatic text depth styling. The Icon block defaults to `core/info` via its `block.json` registration, removing an insert-time effect and adding transform controls to the toolbar group.
- **Data Layer & RT**: Real-time collaboration filters now respect per-post-type disabling configurations. Code editor cursor positioning was patched to prevent jumping during remote sync events.

## Contribution

Released July 1, 2026 as part of the biweekly Gutenberg plugin rollout. The release was driven by a large, coordinated merge effort led by core maintainers including @aduth, @jasmussen, @jorgefilipecosta, and @oandregal, focusing on unifying device previews, stabilizing experimental media features, and expanding Global Styles capabilities alongside extensive TypeScript migration work.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
