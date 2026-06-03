# What’s new in Gutenberg 23.3? (03 Jun)

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Arthur Chu
- **Published:** 2026-06-03
- **Tags:** `General`, `block-editor`, `core-editor`, `gutenberg`, `gutenberg-new`
- **Link:** [https://make.wordpress.org/core/2026/06/03/whats-new-in-gutenberg-23-3-03-jun/](https://make.wordpress.org/core/2026/06/03/whats-new-in-gutenberg-23-3-03-jun/)

## Summary

Gutenberg 23.3 upgrades the editor to React 19 and extends the Block Style States API to per-instance responsive styles, while shifting the default media cropping workflow to a dedicated modal. These changes modernize the editor’s rendering pipeline, improve accessibility across core blocks, and prepare the codebase for long-term framework maintenance without requiring immediate migration from existing patterns.

## Impact

- **Plugin & theme developers**: React 19 deprecates string refs, `ReactDOM.render` fallbacks, and `defaultProps` on function components. Audit custom editor integrations against React’s upgrade notes to prevent runtime warnings or crashes.
- **Block developers**: The Block Style States API now extends to individual block instances, exposing new layout and pseudo-state inspector controls. No action required unless explicitly building custom responsive style logic or overriding state-driven render branches.
- **All users & plugin developers**: The media editor crop tool is now modal-based by default; plugins relying on the previous inline cropping DOM structure may need UI adjustments.
- **Dashboard developers**: The customizable dashboard remains behind an experimental flag (`Gutenberg > Experiments > New Dashboard experience`); no production impact required yet.

## Technical details

- **React 19 Upgrade**: Gutenberg’s build target is now React 19 (Trac #61521). Legacy patterns like string refs and `ReactDOM.render` fallbacks are no longer guaranteed to work; function components must adopt the modern props/default pattern.
- **Block Instance Responsive Styles**: The Block Style States API extends to per-instance level (Trac #76491, #78384). Layout styles and pseudo-states are now applied per-block instance via block supports, with the Inspector conditionally rendering relevant settings when a state is selected (#78280, #78543). State overrides unset grid span defaults and hide layout controls when active (#78709, #78670).
- **Media Editor Modal**: Replaced inline cropping with a modal workflow (Trac #78653), introducing freeform/aspect-ratio cropping, snap rotation, metadata editing, and keyboard-scoped shortcuts. Crop state is now managed via a composite reducer (#78480) and extracts save/crop options hooks (#78225, #78263).
- **Component & DataLayer Shifts**: `__experimentalText`, `__experimentalHStack`, and `__experimentalVStack` are migrated to stable `@wordpress/ui` equivalents (#78155). Real-time Collaboration limits (`/wp-json/wp-sync/v1/updates`) now enforce a 16 MiB payload cap with explicit error handling (#77724), and the data layer avoids duplicate ID-less entity permission requests (#78262).

## Contribution

Released June 3, 2026, as part of the biweekly Gutenberg release cadence. The changes were merged by a cross-functional team including @adamsilverstein, @desrosj, @westonruter, and dozens of first-time contributors. The release coordinated the React 19 upgrade alongside performance optimizations (shared window listeners, optimized CSS parsing) and real-time collaboration reliability fixes. No major alternative approaches were highlighted in the release notes; the merge prioritized stability, accessibility refinements, and incremental rendering optimizations over experimental UI rewrites.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
