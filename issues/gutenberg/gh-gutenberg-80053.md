# #80053: Element: Make the package importable in React 19

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @SirLouen
- **Labels:** `[Type] Enhancement`, `[Package] Element`
- **Merged:** [`190111f`](https://github.com/WordPress/gutenberg/commit/190111f98a9bd0748378521b918a1265d3917779)
- **Discussion:** [#80053](https://github.com/WordPress/gutenberg/pull/80053) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/element` package now supports React 19 by shifting `react` and `react-dom` from `dependencies` to `peerDependencies` and updating its internal import strategy. Previously, the package crashed at load time under React 19 because it used named imports for four `react-dom` APIs that were removed in React 19. This change also widens the React peer dependency ranges across the WordPress design-system packages to accept both React 18 and 19.

## Impact

- **Plugin & theme developers using React:** Must now explicitly install `react` and `react-dom` in their projects, as `@wordpress/element` no longer bundles them.
- **Consumers of `@wordpress/element` or design-system packages:** Will no longer encounter a `SyntaxError` when importing the package in a React 19 environment.
- **Breaking change:** `react`, `react-dom`, `@types/react`, and `@types/react-dom` moved from `dependencies` to `peerDependencies` in `packages/element/package.json`. Consumers must provide these versions themselves (`^18 || ^19`).
- **No action required** for standard WordPress block/theme development unless you are building a custom React application that consumes `@wordpress/element` directly.

## Technical details

The diff modifies `packages/element/package.json` to remove `react`, `react-dom`, and their type definitions from `dependencies` and add them to `peerDependencies` with ranges `^18 || ^19`. The type definitions are marked optional via `peerDependenciesMeta`. In `packages/element/src/react-platform.ts`, the import strategy changes to avoid hard failures when React 19 drops deprecated exports:

**Before:**
```ts
import {
  createPortal,
  findDOMNode,
  flushSync,
  render,
  hydrate,
  unmountComponentAtNode,
} from 'react-dom';
```

**After:**
```ts
import * as ReactDOM from 'react-dom';

const {
  createPortal,
  flushSync,
  findDOMNode,
  render,
  hydrate,
  unmountComponentAtNode,
} = ReactDOM;
```

This namespace import prevents a parse-time `SyntaxError` in React 19, where the removed APIs resolve to `undefined` at runtime instead of failing at import. The `CHANGELOG.md` explicitly marks this dependency shift as a breaking change.

## Contribution

Opened by @SirLouen as a follow-up to #80024, the PR was refined after @simison noted overlap with @aduth’s earlier work. @aduth highlighted the widespread impact of `@wordpress/element` failing under React 19 and requested review from @jsnajdr, who is leading the broader React 19 upgrade effort. The author consolidated the changes after rebasing, and a changelog entry was added during review to formally classify the dependency shift as breaking.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
