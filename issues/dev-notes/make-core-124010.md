# Roadmap to 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** annezazu
- **Published:** 2026-06-19
- **Tags:** `General`, `Updates`, `7.1`, `release-roadmap`
- **Link:** [https://make.wordpress.org/core/2026/06/19/roadmap-to-7-1/](https://make.wordpress.org/core/2026/06/19/roadmap-to-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 plans to enforce an iframe-based editor shell for block-based themes, isolating the editing canvas from the admin DOM. This architectural shift allows viewport-relative units to resolve correctly against the canvas sandbox rather than the browser window, but requires all blocks operating within the editor to declare Block API version 3.

## Impact

- **Theme and plugin developers**: Verify that every custom block shipped with your block-based theme declares `apiVersion: 3` in `block.json`. Blocks using API version 1 or 2 will trigger an automatic fallback to the legacy non-iframed mode, potentially breaking layout assumptions tied to the iframe environment.
- **Extenders and block authors**: Audit any custom styling or JavaScript that assumes direct DOM access or admin-level stylesheet scoping. Viewport-relative units (`vh`, `dvw`) will now calculate against the iframe sandbox, requiring adjustments to full-screen height calculations or position:fixed overlays.
- **Platform and hosting teams**: Note iframe sandbox constraints for any plugins that inject scripts into the editor shell or rely on cross-frame DOM manipulation.
- **Site owners and general users**: No action required; core blocks are pre-migrated and standard theme.json configurations remain unaffected.

## Technical details

The 7.1 rollout enforces an iframe wrapper around the editor shell when a block-based theme is active. Because legacy blocks (Block API version 2 or lower) are incompatible with this isolated environment, the editor automatically degrades back to a non-iframed mode when such blocks are detected. Developers must update `block.json` schemas to declare `apiVersion: 3`, migrate pre-existing block architectures to work within the iframe sandbox, and verify that viewport calculations no longer reference the top-level document. A migration guide is documented to assist extenders in adapting custom block styling and script injection to the iframed context.

## Contribution

This roadmap entry was published by @annezazu on June 19, 2026, synthesizing ongoing development tracked in the associated Trac ticket and dev notes. The decision to initially restrict iframe enforcement to block-based themes reflects a phased rollout strategy agreed upon in the Core Committer meeting, allowing migration validation and ecosystem testing before expanding the shell to all themes in a subsequent release cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
