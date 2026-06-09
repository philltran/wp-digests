# #78923: React: add ReactCurrentOwner polyfill

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jsnajdr
- **Labels:** `[Type] Bug`
- **Merged:** [`c50c8c0`](https://github.com/WordPress/gutenberg/commit/c50c8c0c8b8908770542b5f8d18b00626917e3e0)
- **Discussion:** [#78923](https://github.com/WordPress/gutenberg/pull/78923) · 3 comments · 0 reactions

## Summary

Plugins bundling `framer-motion` (e.g., WooCommerce) crash in the editor with `Uncaught TypeError: Cannot read properties of undefined (reading 'ReactCurrentOwner')`. This PR resolves the failure by injecting a polyfill into the bundled `react` package, preventing runtime errors when third-party libraries reference internal React internals during JSX transformation.

## Impact

- **Plugin & theme developers**: Eliminates immediate editor crashes triggered by `framer-motion` dependencies. No code changes or migration steps are required; updating to the release containing this patch restores functionality.
- **Hosted platforms & headless consumers**: No action required. The polyfill is compiled into the React vendor bundle and does not modify public APIs, REST schemas, or block metadata.

## Technical details

- The change adds a compatibility shim to the `react` package source that safely handles reads from `__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED`. 
- The polyfill assigns the value to the `_owner` property on generated JSX elements, satisfying legacy jsx-runtimes copied from React 18. 
- React 19 discards this field entirely, rendering the shim inert but functionally safe. 
- The compiled diff increases `build/scripts/vendors/react.min.js` by +62 B with no ripple effects to other vendor or core modules.

## Contribution

Opened and merged by @jsnajdr (commit `c50c8c0`). The discussion was brief, focusing on reproducing the crash with bundled `framer-motion` and validating that a lightweight polyfill was the most pragmatic fix. No alternative implementation strategies were debated; the commit landed as a direct compatibility shim.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
