# #79594: Theme: Revert ThemeProvider stable API

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Edit Site`, `[Package] Theme`, `[Package] UI`
- **Merged:** [`55a2cb9`](https://github.com/WordPress/gutenberg/commit/55a2cb9f591df7f2a84709ac32c3e86ed40a3306)
- **Discussion:** [#79594](https://github.com/WordPress/gutenberg/pull/79594) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Reverts the temporary promotion of `@wordpress/theme`'s `ThemeProvider` to a stable public API, restoring it as an internal private boundary. The previous change introduced build failures across core packages that relied on existing private imports, prompting an immediate rollback of the runtime export while preserving documentation and changelog entries.

## Impact

- Plugin & theme developers: No action required. `ThemeProvider` was never intended for external consumption.
- Core/internal package maintainers: Direct named imports from `@wordpress/theme` will break. Consumers must now route through the `privateApis` boundary using `@wordpress/private-apis`.
- Build tooling: Workspace dependencies in `packages/theme/package.json` and `packages/ui/package.json` now require `@wordpress/private-apis`. TS config references are updated accordingly.

## Technical details

The diff rewrites `packages/theme/src/index.ts` to only type-export `ThemeProvider` (`export { type ThemeProvider }`) and adds `export { privateApis } from './private-apis'`. Two new files establish the boundary: `packages/theme/src/lock-unlock.ts` invokes `__dangerousOptInToUnstableAPIsOnlyForCoreModules(...)` for `@wordpress/theme`, and `packages/theme/src/private-apis.ts` locks `ThemeProvider` and `useThemeProviderStyles` into a `privateApis` object. Consumers across `@wordpress/boot` (`root/index.tsx`, `single-page.tsx`), `@wordpress/edit-site` (`layout/index.js`), `@wordpress/ui` (all `popup.tsx` components), and Storybook decorators now import `{ privateApis as themePrivateApis } from '@wordpress/theme'` and instantiate the component via `unlock(themePrivateApis).ThemeProvider`. Corresponding `package.json` and `tsconfig` files are updated to include `@wordpress/private-apis` and project references.

## Contribution

Opened and merged by @ciapo (co-authored by @tyxla) as PR #79594 to resolve build errors caused by #78958. The change restores the private API boundary across internal packages while preserving non-runtime documentation. Cherry-picked to `release/23.5` and `wp/latest` branches for inclusion in npm releases `@wordpress/theme@0.17.0` and `@wordpress/ui@0.16.1`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
