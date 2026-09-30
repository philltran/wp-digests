# #82154: Build: Fix regular CSS injection in Vitest Browser Mode

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] wp-build`
- **Merged:** [`cd9c906`](https://github.com/WordPress/gutenberg/commit/cd9c906e022018899e1e25addaa3f95368e180a5)
- **Discussion:** [#82154](https://github.com/WordPress/gutenberg/pull/82154) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/build` now emits a browser-safe injection guard for regular (non-CSS-Module) CSS, matching what CSS Modules already used. Previously the regular CSS output only checked `process.env.NODE_ENV !== 'test'`, and Vitest Browser Mode sets `NODE_ENV=test`, so those styles were never injected in Chromium. Node and jsdom still skip injection in test mode, while Browser Mode, which has no `process`, now receives styles. The CSS compiler was also extracted into its own module so it can be tested directly.

## Impact

- **Plugin/theme developers using `@wordpress/build` with Vitest Browser Mode:** regular `.css`/`.scss` output will now apply in Chromium tests, so computed-style assertions work without workarounds.
- **Action required:** packages must be rebuilt to pick up the new guard. Already generated files keep the old behavior.
- **Node/jsdom test setups:** no change. Injection is still skipped when `process` is defined and `NODE_ENV === 'test'`.
- **Production and normal browser builds:** no behavior change. The guard still injects whenever `NODE_ENV` is not `test` or `process` is undefined.
- **Bundle size:** the generated guard adds a few bytes per CSS-containing module. The CI size report showed a small increase (+28 B in the final run).
- No deprecations or API removals.

## Technical details

The diff moves `compileInlineStyle()` and the optional `@wordpress/theme/postcss-plugins/postcss-ds-token-fallbacks` import out of `packages/wp-build/lib/build.mjs` into a new `packages/wp-build/lib/compile-inline-style.mjs`. The token-fallback plugin remains optional, so the import still fails gracefully when `@wordpress/theme` is absent. `build.mjs` imports `compileInlineStyle` and `dsTokenFallbacks` from the new module. The esbuild JS token-fallback plugin stays in `build.mjs`.

Both output paths now share one condition string:

```js
const shouldInjectStyle =
  `typeof process === 'undefined' || process.env.NODE_ENV !== 'test'`;
```

Before (regular CSS):

```js
if (typeof document !== 'undefined' && process.env.NODE_ENV !== 'test' && !document.head.querySelector("style[data-wp-hash='…']")) { … }
```

After:

```js
if (typeof document !== 'undefined' && (typeof process === 'undefined' || process.env.NODE_ENV !== 'test') && !document.head.querySelector("style[data-wp-hash='…']")) { … }
```

The CSS Modules path (`registerStyle` from `@wordpress/style-runtime`) now uses the same shared condition, and its semantics are unchanged. The old regular-CSS guard would also have thrown a `ReferenceError` on a bare `process.env` access where `process` is undefined and `NODE_ENV` was not short-circuited; the new `typeof` check avoids that.

Tests and config:
- `lib/test/compile-inline-style.test.js` runs generated modules through `new Function` with a fake `document` and `process`. It covers token fallbacks, skipping in test mode, injecting in development, and injecting when `process` is absent.
- `style-injection.browser.test.js` (Chromium) checks computed styles for both CSS Module and ordinary CSS output.
- `style-injection.jsdom.test.js` asserts that nothing is injected in jsdom.
- `test/unit/vitest.config.mjs` adds a `wpBuildStyleFixturePlugin` that serves `virtual:wp-build-style-injection` and `virtual:wp-build-ordinary-style-injection` from the real `compileInlineStyle` output. The PR description also says the routing check now recognizes Vitest's `browser (chromium)` project name.
- `.gitignore` adds `/.vitest-attachments/` and `**/__screenshots__/*.browser.test.*/`.
- The README's "Testing Generated Styles" section and the `wp-build` CHANGELOG were updated.

## Contribution

This follows up on #80855 and resolves a Browser Mode compatibility issue anticipated in an earlier comment on #75792. #80998 is stacked on this PR, and #82574 contains the same Browser Mode routing-label normalization, so whichever merged second was to drop the overlap. The author notes Codex assisted with implementation and verification. Discussion in the record is limited to bot reports (size, flaky e2e tests, props).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
