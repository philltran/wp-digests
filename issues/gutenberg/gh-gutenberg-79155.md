# #79155: Core Abilities: Restore the ready promise and lazy-load via dynamic import

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Bug`, `Backport to Gutenberg RC`, `Backported to WP Core`, `No Core Sync Required`, `[Package] Abilities`
- **Merged:** [`7ba9f39`](https://github.com/WordPress/gutenberg/commit/7ba9f3949e3ca7c5abd48bd30143322d8655cee2)
- **Discussion:** [#79155](https://github.com/WordPress/gutenberg/pull/79155) · 10 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

PR #79155 restores the public `ready` promise and auto-initialization contract for `@wordpress/core-abilities` after PR #78316 inadvertently removed it. The change reverts the forced `initialize()` function, allowing the module to automatically fetch and register abilities on import while deferring network requests via dynamic `import()`. This restores backward compatibility without sacrificing the performance gains from deferred loading.

## Impact

- **Plugin & theme developers**: No action required. The `initialize()` function is removed, and the `ready` promise is restored. Consumers relying on the previous API contract will work as expected.
- **Core/Platform**: Removes the unconditional `wp_enqueue_script_module( '@wordpress/core-abilities' )` from `lib/client-assets.php`, preventing unnecessary `/wp-abilities/v1/*` REST requests on every admin page load.
- **Breaking changes**: None. The public API surface is restored to its pre-#78316 state.

## Technical details

- `packages/core-abilities/src/index.ts`: The public `initialize()` export is removed and replaced with an internal `initialize()` function that runs immediately upon module evaluation. The resolved promise is exported as `ready`.
- `lib/client-assets.php`: The `gutenberg_enqueue_core_abilities()` callback and its `admin_enqueue_scripts` hook attachment are removed, stopping the unconditional script module enqueue.
- `packages/workflow/src/components/workflow-menu.js`: Replaces the synchronous `initializeCoreAbilities()` call with a dynamic `import( '@wordpress/core-abilities' )` inside the `useEffect` that triggers when the workflow palette opens.

**Usage pattern change:**
```js
// Before (broken by #78316)
import { initialize } from '@wordpress/core-abilities';
await initialize();

// After (restored)
import { ready } from '@wordpress/core-abilities';
await ready;
```

The module registry still registers `@wordpress/core-abilities`, but the network requests only fire when the dynamic import executes, leveraging the ESM module cache to ensure fetches run at most once.

## Contribution

Opened and merged by @jorgefilipecosta with co-authors @t-hamano and @ellatrix. The PR was created to address a backward compatibility regression introduced in #78316. Review focused on clarifying when network requests fire and confirming the dynamic import pattern aligns with existing loader patterns like `@wordpress/vips/loader`. The approach was accepted to restore the public API contract before the Gutenberg RC cutoff.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
