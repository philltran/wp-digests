# Dev Chat summary: July 1, 2026

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Jb Audras
- **Published:** 2026-07-01
- **Tags:** `Devchat`, `General`, `Summaries`, `7.0`, `7.0.1`, `7.1`, `core`, `dev chat`
- **Link:** [https://make.wordpress.org/core/2026/07/01/summary-dev-chat-july-1-2026/](https://make.wordpress.org/core/2026/07/01/summary-dev-chat-july-1-2026/)
- **Usefulness:** 3/5

## Summary

WordPress 7.1 development has formalized a roadmap proposal to hide the Classic Block from the block editor inserter by default. This shifts legacy text editing from an opt-out to an opt-in model, preserving shortcode and `the_content` rendering capabilities while prioritizing modern block-based authoring surfaces.

## Impact

- **Site owners/editors**: The Classic Block will be absent from the default inserter dropdown; enabling it requires adjusting editor preferences or site settings.
- **Plugin & theme developers**: Audit fallback rendering for shortcodes and legacy content. Update integration tests that assume `classic` block availability by default.
- **Core ecosystem**: Aligns future dev notes and deprecation guidance around legacy block visibility. No immediate code changes are required during the beta cycle.

## Technical details

This is listed as a roadmap proposal for the 7.1 release cycle rather than an applied patch. The change targets the default visibility state of the `classic` block in the editor registry, controlling inserter availability via site settings or editor preferences rather than hardcoding registry state. Specific implementation specifications and developer-facing documentation will be published as dev notes throughout the beta phase.

## Contribution

Formally added to the 7.1 roadmap agenda during the July 1, 2026 Dev Chat. Core contributors reviewed it alongside parallel proposals (e.g., a new custom post type proposal) and agreed to advance it as a guiding release principle. Detailed technical specifications and migration documentation will be tracked in upcoming dev notes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
