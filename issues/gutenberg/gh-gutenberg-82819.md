# #82819: Commands: Avoid re-registering command loaders on every render

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Commands`
- **Merged:** [`859c595`](https://github.com/WordPress/gutenberg/commit/859c595b481b63f4ebbc9fcf711bb1cd77396cbc)
- **Discussion:** [#82819](https://github.com/WordPress/gutenberg/pull/82819) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`useCommandLoader` in `@wordpress/commands` no longer unregisters and re-registers its command loader whenever the `hook` option changes identity. Callers commonly build `hook` during render, so the registration effect previously fired on every render of the calling component. The hook is now stored in a ref and a stable wrapper is registered instead, so the loader registers once.

## Impact

- **Plugin & theme developers using `useCommandLoader`:** No code changes are required. Loaders that pass an inline or per-render `hook` now register once instead of churning the commands store on every render.
- **Behavior change to be aware of:** The palette always calls the most recent `hook`. Code that (perhaps unintentionally) relied on a changed `hook` identity triggering unregister/re-register will no longer get that. A reviewer flagged this as potentially breaking for such callers, and a note was added to the JSDoc/README.
- **Site owners / end users:** No user-facing change. The command palette should behave as before.
- **Hosting, headless & REST consumers:** Not affected.

## Technical details

The change is in `packages/commands/src/hooks/use-command-loader.js`, and mirrors the pattern `useCommand` already uses for its `callback` option.

- `currentHookRef = useRef( loader.hook )` holds the latest hook, updated in a `useEffect` keyed on `loader.hook`.
- A `useCallback` with empty deps creates a stable wrapper: `( ...args ) => currentHookRef.current( ...args )`.
- `registerCommandLoader` now receives this stable `hook` instead of `loader.hook`, and the registration effect's dependency array swaps `loader.hook` for the stable `hook`. The remaining deps are `loader.name`, `loader.context`, `loader.category`, and `loader.disabled`.

```js
// Before: effect re-ran whenever loader.hook identity changed
registerCommandLoader( { name, hook: loader.hook, context, category } );

// After: stable wrapper, latest hook read from a ref at call time
const hook = useCallback( ( ...args ) => currentHookRef.current( ...args ), [] );
registerCommandLoader( { name, hook, context, category } );
```

The ref is updated in an effect, not during render. The README and JSDoc gain the note: "The palette always calls the most recent `hook`. Changing the `hook` instance doesn't re-render the palette." A `CHANGELOG.md` entry is added under Unreleased → Bug Fixes. The bundle size of `build/scripts/commands/index.min.js` grows by 27 B (+0.14%). No hooks, REST schema, or data changes.

## Contribution

@Mamaduka authored the PR (noting it was assisted by Claude). @youknowriad raised that the change could be breaking for anyone relying on the old re-registration behavior and asked for it to be documented in the `useCommandLoader` docs. Mamaduka added the JSDoc/README note, and the PR was merged after that.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
