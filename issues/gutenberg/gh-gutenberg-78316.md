# #78316: Core Abilities: Defer fetch until workflow palette opens

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Performance`, `[Package] Commands`
- **Merged:** [`80c5c88`](https://github.com/WordPress/gutenberg/commit/80c5c8816f47b265eb283538249f62c964ee60c4)
- **Discussion:** [#78316](https://github.com/WordPress/gutenberg/pull/78316) · 4 comments · 0 reactions

## Summary

Defers two sequential REST API requests (`GET /wp-abilities/v1/categories` and `GET /wp-abilities/v1/abilities`) from immediate page-load execution to on-demand triggering when the workflow palette modal opens. This eliminates unnecessary HTTP requests during wp-admin bootstrap, improving editor performance for features currently gated behind the `gutenberg-workflow-palette` experiment.

## Impact

- **Experimental package consumers**: The `@wordpress/core-abilities` package removes its previous synchronous top-level execution and `ready` promise contract in favor of an explicitly called memoized `initialize()` function. Any code relying on the prior auto-initialization pattern must be updated to call the new function imperatively.
- **Workflow palette consumers**: No action required. The workflow menu component now automatically handles initialization lifecycle when the modal mounts.
- **Core/Plugin developers**: No breaking changes outside the experimental `lib/experimental/` namespace; all affected packages remain pre-1.0.

## Technical details

- `packages/core-abilities/src/index.ts` replaces the eager top-level module execution with an exported memoized `initialize()` function that caches fetch results across calls.
- `packages/workflow/package.json` explicitly declares `@wordpress/core-abilities` as a dependency to ensure correct bundle resolution order during compilation.
- `packages/workflow/src/components/workflow-menu.js` invokes the new `initialize()` function within a `useEffect` hook triggered upon palette open, preventing redundant fetches on subsequent modal openings. The previous exported `ready` promise is removed as the contract remains explicitly unstable.

## Contribution

Opened and merged by @ellatrix (commit `80c5c88`). The PR addresses an early-review observation that REST requests were firing eagerly on every admin page load despite the feature being hidden behind an experiment flag. The final diff locks in a deferred, memoized initialization pattern paired with explicit package dependency declaration, rejecting further architectural changes during the merge window.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
