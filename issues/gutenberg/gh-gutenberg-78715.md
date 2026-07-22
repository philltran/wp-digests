# #78715: wp-build: Return null from getPackageInfo on resolve miss instead of throwing

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @chihsuan
- **Labels:** `[Type] Bug`, `[Package] wp-build`
- **Merged:** [`bc0b1ab`](https://github.com/WordPress/gutenberg/commit/bc0b1abeeca07c91e82e9e247d05b899c28e3cb3)
- **Discussion:** [#78715](https://github.com/WordPress/gutenberg/pull/78715) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

@wordpress/build's `getPackageInfo()` function now returns `null` instead of throwing when a module cannot be resolved. This aligns the implementation with its documented `@return {PackageJson|null}` contract and prevents build crashes when projects use `tsconfig` `paths` to map namespaced imports that aren't installed in `node_modules`.

## Impact

- **Plugin & theme developers**: No action required for standard setups. If your build previously failed with `Cannot find module '@your-namespace/.../package.json'` due to `tsconfig` `paths` mappings, this fix resolves the crash.
- **Build tooling & platform teams**: The `wordpress-externals` esbuild plugin now correctly falls through to inline bundling for unresolvable specifiers, matching the existing behavior of `inferStyleDependencies`.

## Technical details

The diff wraps the `require.resolve()` call inside `getPackageInfo()` in a `try/catch` block. It explicitly treats `MODULE_NOT_FOUND` and `ERR_PACKAGE_PATH_NOT_EXPORTED` as resolution misses, returning `null` and caching the negative result to avoid repeated throws. Any other error is rethrown to prevent silent failures. This change activates dormant `null` guards in the `wordpress-externals` plugin (`packages/wp-build/lib/wordpress-externals-plugin.mjs`) and `inferStyleDependencies` (`packages/wp-build/lib/build.mjs`), allowing esbuild to resolve the specifier via its own `tsconfig` `paths` discovery and bundle it inline.

## Contribution

Opened by @chihsuan and merged with co-authors @retrofox and @youknowriad, the PR directly addresses a contract violation where the implementation threw instead of returning `null`. The author noted that automated testing was skipped due to `@wordpress/build`'s `tsconfig.json` lacking test-runner globals, opting for manual verification instead. The fix was reviewed and merged without significant design debate, as it simply enforces the existing JSDoc promise.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
