# #82189: Build: Require explicit JSX file extensions

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Breaking Change`, `[Package] wp-build`
- **Merged:** [`3ebc58f`](https://github.com/WordPress/gutenberg/commit/3ebc58f15270c50b9c326ec7c674547166562139)
- **Discussion:** [#82189](https://github.com/WordPress/gutenberg/pull/82189) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/build` no longer parses JSX in `.js` source files. The four `loader: { '.js': 'jsx' }` esbuild overrides used during package and worker-code transpilation were removed, so JSX must now live in `.jsx` or `.tsx` files. This aligns the tool with esbuild's default behavior and with Gutenberg's own explicit-extension convention. It is labeled a breaking change.

## Impact

**Plugin & theme developers using `@wordpress/build`**
- **Breaking:** any source file that contains JSX but has a `.js` extension will no longer be parsed as JSX. Rename such files to `.jsx` or `.tsx` (and update imports that include the extension) before building.
- Files without JSX, and files already using `.jsx`/`.tsx`, are unaffected.

**Gutenberg contributors**
- No action required; the repo already uses explicit JSX extensions (per the follow-up to #80990), and the full `npm run build` is the stated verification.

**Site owners, hosting, headless/REST consumers**
- No action required; this is build-tooling only.

## Technical details

The diff removes `loader: { '.js': 'jsx' }` from four `esbuild` build option objects:

- `packages/wp-build/lib/build.mjs`: two occurrences in `transpilePackage()`.
- `packages/wp-build/lib/worker-build.mjs`: two occurrences in the worker-code transpilation that generates the `workerCode` export.

The `jsx: 'automatic'` and `jsxImportSource: 'react'` options remain, so `.jsx`/`.tsx` files still use the automatic React runtime. With the override gone, esbuild falls back to its default loader for `.js` (plain JS), so JSX syntax in a `.js` file will fail to parse.

```js
// before
esbuild.build( { jsx: 'automatic', jsxImportSource: 'react', loader: { '.js': 'jsx' }, plugins } )
// after
esbuild.build( { jsx: 'automatic', jsxImportSource: 'react', plugins } )
```

`packages/wp-build/CHANGELOG.md` gains a new "Breaking Changes" entry under Unreleased documenting the rename requirement.

## Contribution

Authored by @ciampo as a follow-up to #80990, with @aduth credited in the props-bot list. The record shows no design debate; the discussion consists only of bot comments (size check, props, and a flaky e2e test report unrelated to the change). The PR notes Codex assisted with implementation, documentation, and verification.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
