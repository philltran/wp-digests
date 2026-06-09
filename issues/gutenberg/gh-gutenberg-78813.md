# #78813: Remove migrated dependencies from root package.json

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @kushagra-goyal-14
- **Labels:** `[Type] Code Quality`
- **Merged:** [`d0aff7d`](https://github.com/WordPress/gutenberg/commit/d0aff7d0df29c76ba0fabbbc77ee6c570b590851)
- **Discussion:** [#78813](https://github.com/WordPress/gutenberg/pull/78813) · 7 comments · 0 reactions

## Summary

Cleans up the Gutenberg monorepo’s root package.json by stripping devDependencies that are no longer consumed at the repository root level following a prior migration. Several Webpack-era loaders, build utilities, and TypeScript type definitions remained redundantly in the root workspace. This change removes those entries and relocates necessary type definitions to their appropriate consumer packages, reducing dependency footprint and eliminating stale tooling configurations.

## Impact

- **Site owners & end users:** No action required. This is strictly an internal build-toolchain cleanup; released Gutenberg packages and editor runtime behavior remain unchanged.
- **Plugin & theme developers:** No impact. External dependencies do not consume the monorepo's root package.json.
- **Core contributors & maintainers:** Running npm install at the repository root will no longer resolve css-loader, postcss-loader, raw-loader, sass, sass-embedded, sass-loader, source-map, style-loader, @types/node, or @types/prettier. If you maintain a vendored fork of the monorepo, ensure build scripts that previously relied on root-level types explicitly reference packages/prettier-config/package.json or declare their own devDependencies.

## Technical details

- Modifies `package.json` in the monorepo root: removes css-loader, postcss-loader, raw-loader, sass, sass-embedded, sass-loader, source-map, style-loader, @types/node, and @types/prettier from devDependencies.
- Updates `packages/prettier-config/package.json`: adds @types/node and @types/prettier to its devDependencies block.
- Modifies `test/performance/tsconfig.json` compilerOptions by removing the "typeRoots" array, shifting TypeScript resolution to standard paths.
- The diff confirms these are purely static analysis and build-time adjustments. No PHP files, block registration scripts, REST schema definitions, or JavaScript/TypeScript runtime logic were modified.

## Contribution

Merged as d0aff7d by @kushagra-goyal-14 following review from @manzoorwanijk. Initially scoped to cleaning up root-level Webpack-era Storybook leftovers, the scope expanded during the PR to include redundant type definitions and other untracked dev tools. The final diff reflects a strict removal/relocation pattern rather than structural package restructuring, aligning with the broader monorepo cleanup effort.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
