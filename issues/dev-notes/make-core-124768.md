# React 19: punted beyond WordPress 7.1, experiment in Gutenberg

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Jarda Snajdr
- **Published:** 2026-07-24
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/07/24/react-19-punted-beyond-wordpress-7-1-experiment-in-gutenberg/](https://make.wordpress.org/core/2026/07/24/react-19-punted-beyond-wordpress-7-1-experiment-in-gutenberg/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 will ship with React 18.3, deferring the React 19 upgrade to a future release. An initial attempt to upgrade was reverted after testing revealed incompatibilities when React 18 and 19 coexist in the same environment, particularly with how plugins bundle and consume React. Gutenberg 23.4+ now includes an experimental flag to enable React 19 for plugin compatibility testing.

## Impact

- **Plugin & theme developers:** No immediate action required for WordPress 7.1, as React 18.3 remains the default. Developers enabling the Gutenberg experiment must audit their code for React 19 removals (string refs, default props on function components, legacy context patterns) and ensure they do not bundle `react/jsx-runtime` directly.
- **Hosting & platform teams:** No configuration changes needed for 7.1. When React 19 ships, environments will need to verify that externalized React scripts are correctly served and that legacy plugin code does not conflict with the new runtime.
- **Headless & REST consumers:** Unaffected; the React version change is confined to the block editor and admin UI.

## Technical details

The change reverts a previously attempted React 19 upgrade in core, maintaining `react` and `react-dom` at version `18.3` in WordPress 7.1. Gutenberg 23.4+ introduces an experiment (enabled via the Gutenberg Experiments settings page) that swaps the core React dependency to version 19. The primary technical friction stems from plugins that inline `react/jsx-runtime` in their build output, causing dual-runtime conflicts where React 18 and 19 instances exchange incompatible internal data structures. The Gutenberg compat layer previously polyfilled several deprecated React 18 features that were fully removed in React 19, including string refs, default props on function components, and legacy context API patterns. Plugin authors should verify their build configurations externalize React dependencies to match WordPress's provided scripts rather than bundling them.

## Contribution

The decision to defer the upgrade followed a brief enablement period in Gutenberg where dual-runtime conflicts and plugin incompatibilities surfaced, prompting a revert. The experimental flag was added in Gutenberg 23.4 to allow the community to test compatibility ahead of a core release. Key contributors to the revert, experiment implementation, and review include @tyxla, @aduth, @simison, @wildworks, and @mamaduka. The team is coordinating with the Plugin Check project to automate detection of these bundling and API usage issues.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
