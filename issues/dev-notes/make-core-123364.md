# Media Editor Modal: Call for Testing

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** ramonopoly
- **Published:** 2026-05-21
- **Tags:** `General`, `Proposals`, `core-editor`, `gutenberg`, `media`, `media modal`
- **Link:** [https://make.wordpress.org/core/2026/05/21/media-editor-modal-call-for-testing/](https://make.wordpress.org/core/2026/05/21/media-editor-modal-call-for-testing/)

## Summary

The Block Editor now replaces its legacy inline image cropper with a new Media Editor Modal. This shift consolidates freeform and aspect-ratio cropping, flip operations, snap rotation, and metadata editing into a single modal workflow. The change eliminates reliance on the third-party `react-easy-crop` library in favor of a WordPress-native component collection, providing a more extensible foundation for image editing across editor contexts.

## Impact

- **Block & Plugin Developers:** The DOM structure and JavaScript dependencies for the Image block's cropping flow have changed. Any custom scripts, styles, or event listeners targeting the previous inline cropper must be updated to integrate with the new modal surface.
- **Theme Developers:** No direct changes required unless themes override Image block crop controls or interact directly with the legacy modal's DOM.
- **Editors & Site Owners:** The crop workflow now launches as a dedicated modal rather than an inline canvas. Standard cropping, rotation, flip, and metadata updates function seamlessly. No manual configuration, experiment toggles, or migration steps are required.

## Technical details

The Media Editor Modal supersedes the inline cropping tool previously built on `react-easy-crop`. It introduces a unified interface handling freeform/aspect-ratio cropping, fine-grained snap rotation, flip operations, and attachment metadata editing (alt text, caption, author, attached-to). PR #78653 merged this implementation to make the modal the default Image block crop flow, removing all experiment toggle requirements. The underlying code abstracts these capabilities into a reusable WordPress-native component collection, ensuring consistent keyboard navigation (`Tab`, `Arrow` keys), touch gesture support (pinch zoom, drag pan), and correct state handling for unsaved changes via `Escape`. Standard REST or DB fields are not altered; the modal directly updates attachment metadata and block attributes upon save.

## Contribution

The feature was initially called for testing via a Make Core post published on 2026-05-21. Following community validation of basic crop flows, keyboard/touch interactions, and metadata editing behavior, PR #78653 was merged to make the Media Editor Modal the default experience across all editor contexts. The tracking post and testing coordination were supported by @andrewserong and @isabel_brison. Development continues on the associated Gutenberg tracking issue for follow-up refinements, with explicit out-of-scope items (manual pixel controls, restore original image, undo/redo history) deferred to future cycles.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
