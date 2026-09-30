# #82144: ESLint plugin: remove usage of Babel parser

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jsnajdr
- **Labels:** `[Type] Bug`, `[Tool] ESLint plugin`
- **Merged:** [`6c8b09e`](https://github.com/WordPress/gutenberg/commit/6c8b09efceb5fb20fd8e154b15c7b92e6e57fe1a)
- **Discussion:** [#82144](https://github.com/WordPress/gutenberg/pull/82144) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `esnext` flat config in `@wordpress/eslint-plugin` no longer sets `@babel/eslint-parser` as its parser, so JS/JSX files are parsed by ESLint's default parser (espree). This removes a scope-analysis bug specific to the Babel parser that failed to process JSX elements and caused false positives in `no-unused-vars-before-return` (#55552). The package also drops its `@babel/eslint-parser`, `@wordpress/babel-preset-default` and `@babel/core` (peer) dependencies.

## Impact

- **Plugin/theme developers using `@wordpress/eslint-plugin`:**
  - JS and JSX are now parsed by espree instead of `@babel/eslint-parser`. The changelog lists this under *Breaking Changes*.
  - Syntax that only the Babel parser (with `@wordpress/babel-preset-default`) accepted, such as proposal-stage or Babel-specific syntax, may now raise parsing errors.
  - The `no-unused-vars-before-return` false positive on variables used only in JSX is fixed.
  - Your own `babel.config.json` is no longer consulted by the lint config.
- **Dependency/install footprint:** `@babel/core` is removed from `peerDependencies`, and `@babel/eslint-parser` and `@wordpress/babel-preset-default` are removed from `dependencies`. Projects that relied on these being installed transitively through the ESLint plugin should declare them directly.
- **Legacy eslintrc users (deprecated `eslintrc/` entry point):** the converter now emits explicit `ecmaVersion: 'latest'`, `sourceType: 'module'` and ES builtin globals, so behavior stays consistent with flat config.
- **Hosting/headless:** not affected.
- Otherwise no action is required, but run lint after upgrading to catch any new parse errors.

## Technical details

**`configs/esnext.js`**: the `languageOptions` block is removed entirely: parser, `parserOptions` (`sourceType: 'module'`, and the `requireConfigFile`/`babelOptions` preset fallback), and `globals.es2015`. The `cosmiconfig` babel-config search and `globals` import are gone. Only the rules remain, layered on `es5Config`.

**`configs/babel-parser-compat.js`** is deleted. This was the wrapper that patched `scopeManager.addGlobals()` for ESLint 10 compatibility.

**`eslintrc/index.js`** (`flatToEslintrc`): the `@wordpress/babel-eslint-parser-compat` entry is removed from `PARSER_NAME_TO_ESLINTRC`. Since eslintrc defaults to `ecmaVersion: 5` / `sourceType: 'script'` and does not derive ES globals from `ecmaVersion`, the converter now always emits them. This came from review feedback by @manzoorwanijk, who showed that without it `import`/arrow/`const` in `.js` gave `Parsing error`, and `new Map()` gave a `no-undef` error.

```js
const esGlobals = require( 'globals' ).builtin;
// ...
globals: { ...esGlobals, ...globals },
parserOptions: {
	ecmaVersion: 'latest',
	sourceType: 'module',
	...parserOptions,
},
```

Spread order lets explicitly configured `sourceType`/`ecmaVersion` win. The previous conditional `globals`/`parserOptions` assignments are removed. `.ts` handling is unchanged (the `typescript-eslint/parser` override).

**`package.json`/lockfile**: removes `@babel/eslint-parser`, `@wordpress/babel-preset-default`, and peer `@babel/core`, plus their transitive lockfile entries (`@nicolo-ribaudo/eslint-scope-5-internals`, `eslint-visitor-keys@2.1.0`).

## Contribution

@jsnajdr found the root cause while exploring a fix for #55552 and an alternative to #80666: the Babel parser's scope analysis mishandles JSX, while the espree and TypeScript parsers don't. He flagged the `flatToEslintrc` ESLint 9 compatibility helper as an open question. @manzoorwanijk reproduced the breakage against the branch and proposed the converter fix, which was adopted. @jsnajdr declined to add a test for the eslintrc path because it would need an aliased ESLint 9 dev dependency, and dropped a README paragraph about TS parsing as unnecessary. Merging the `es5` and `esnext` configs was suggested as a follow-up.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
