# Text Shadow Support in Global Styles

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Aaron Robertshaw
- **Published:** 2026-07-23
- **Tags:** `General`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/07/23/text-shadow-support-in-global-styles/](https://make.wordpress.org/core/2026/07/23/text-shadow-support-in-global-styles/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 adds native `textShadow` support to `theme.json`, allowing developers to declare CSS `text-shadow` values directly in Global Styles without custom CSS or plugins. The property is recognized under `styles.typography`, per-block `styles.blocks.<block>.typography`, and element states like `:hover`. This delivers the foundational layer for typography shadows, with a UI and presets deferred to a subsequent release.

## Impact

- **Theme & plugin developers:** Can now control text shadows declaratively in `theme.json` without enqueuing custom stylesheets.
- **Site owners:** No immediate impact; configuration remains theme-level only until the next release adds editor controls.
- **Hosting & platform teams:** No action required. The change is strictly additive and maintains full backwards compatibility. Existing themes and blocks render identically if `textShadow` is unset.

## Technical details

The change introduces a new `textShadow` property recognized by the Global Styles parser under `styles.typography`, `styles.blocks.<block>.typography`, and `styles.elements.<element>`. Values map directly to the CSS `text-shadow` property, supporting standard syntax including multiple comma-separated shadows. The parser applies block-level values over global values following the existing typography cascade. Additionally, the editor automatically resets `text-shadow` to `none` on the empty rich text placeholder to maintain readability, while actual content retains the configured shadow. No new hooks, filters, or block supports are introduced in this release; the feature is purely style-driven via `theme.json`.

## Contribution

The feature originated from issue #47904 as a long-requested typography enhancement. To unblock progress while UI and preset architecture were still being debated, the core team shipped a minimal, style-only implementation in PR #73320. Review focused on ensuring the property integrated cleanly with the existing `theme.json` typography cascade and editor placeholder handling, with full UI, presets, and `supports.typography.textShadow` deferred to a follow-up PR (#79584). The record contains no detailed design debate beyond the stated decision to defer non-essential components.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
