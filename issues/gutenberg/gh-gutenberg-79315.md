# #79315: Plugins API: Fix the plugin 'render' property validation

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @sdnunca
- **Labels:** `[Type] Bug`, `[Package] Plugins`, `First-time Contributor`
- **Merged:** [`4429bf0`](https://github.com/WordPress/gutenberg/commit/4429bf0841927c58c18da3e7f8e8981285c0332b)
- **Discussion:** [#79315](https://github.com/WordPress/gutenberg/pull/79315) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`registerPlugin()` in `@wordpress/plugins` validated its `render` setting with `typeof render !== 'function'`, which rejects any valid React element type that isn't a plain function — most notably `memo()`-wrapped components. Since #78674 made `withSelect` return a memoized component, every plugin passing a `withSelect`-wrapped component as `render` started failing registration with `The "render" property must be specified and must be a valid function.` and silently not rendering. This PR swaps the check for `isValidElementType()` from the `react-is` package, so memo, forwardRef, class components, and other legitimate element types pass. The console error string was also reworded to say "valid component" instead of "valid function".

## Impact

- **Plugin & extender developers:** If you register an editor plugin whose `render` is wrapped in `memo()`, `withSelect`, or another HOC that returns a memo object, registration works again — no code change on your side. Previously the only workaround was to wrap the memoized component in a plain function component.
- **Anyone asserting on the error text:** The message emitted for an invalid `render` changed from `'The "render" property must be specified and must be a valid function.'` to `'The "render" property must be specified and must be a valid component.'`. Snapshot or `toHaveErroredWith` assertions on the old string will fail.
- **Package consumers:** `@wordpress/plugins` gains a runtime dependency on `react-is` (`^18.3.0`); bundle-size- or dependency-audit-sensitive builds should note the new transitive package.
- **Site owners:** No action required — this is a regression fix that ships with Gutenberg.

## Technical details

In `packages/plugins/src/api/index.ts`, `registerPlugin()` now imports `isValidElementType` from `react-is` and guards on it:

```js
// Before
if ( typeof render !== 'function' ) {
	console.error(
		'The "render" property must be specified and must be a valid function.'
	);
	return null;
}

// After
if ( ! isValidElementType( render ) ) {
	console.error(
		'The "render" property must be specified and must be a valid component.'
	);
	return null;
}
```

`packages/plugins/package.json` adds `react-is: ^18.3.0` to `dependencies` and `@types/react-is: ^18.3.1` to `devDependencies`; the lockfile hoists a `react-is@18.3.1` under `packages/plugins/node_modules/react-is` and re-nests the pre-existing dev-only `react-is@17.0.2` under `pretty-format`. A new test in `packages/plugins/src/api/test/index.js` registers a plugin whose `render` is `memo( Component )` and asserts `getPlugin( name )` returns it; the existing "fails to register a plugin without a render function" test was updated to the new message. No public API signature, hook, or type change — `render` is still typed `ComponentType`.

## Contribution

This is the plugins-API counterpart to PR #57193, which made the same `typeof` → `isValidElementType` swap for the blocks API; the regression itself came from #78674, which changed `withSelect` to return a memoized component. In review, @Mamaduka asked for a test that would fail without the fix, and the author added a dedicated memo case alongside the existing plain-component test rather than modifying the original assertion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
