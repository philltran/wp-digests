# #79552: theme: Validate npm publish surface

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Theme`
- **Merged:** [`985526d`](https://github.com/WordPress/gutenberg/commit/985526d0e44005888077286eb5a5d64f24e6cf90)
- **Discussion:** [#79552](https://github.com/WordPress/gutenberg/pull/79552) · 14 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/theme` npm package publish surface has been cleaned and its ESM declaration export paths corrected. A new validation script now runs during CI to ensure packed tarballs exclude test, story, and snapshot artifacts, and that all local `exports` targets are actually included in the published package.

## Impact

- **Gutenberg core maintainers:** Builds will now fail if `@wordpress/theme` includes disallowed files or missing export targets, preventing broken publishes.
- **Plugin & theme developers:** No immediate code changes required. Consumers of `@wordpress/theme` will receive a smaller, cleaner package with correctly resolved ESM types.
- **Platform/CI:** The `lint:package-contents` script is now part of the Static Analysis workflow and can be opted into by other workspaces via `@wordpress/validation-tools`.

## Technical details

- `packages/theme/package.json` updates the `files` array to explicitly include `build-module`, `build-types`, `src/*-plugins/**`, `src/prebuilt/css`, `src/prebuilt/js`, and `*.md`, while excluding `.d.mts.map`, `.d.ts.map`, `stories/**`, and `test/**`.
- The legacy `"types": "build-types"` field is removed in favor of the `exports` map for type resolution.
- A new CLI tool `wp-validate-package-contents` (sourced from `tools/validation/validate-package-contents.mjs`) is added to `@wordpress/validation-tools`. It runs `npm pack --dry-run --json`, filters packed paths against regex patterns for test/story/snapshot directories, and cross-references `package.exports` local targets against the packed file set.
- Root `package.json` adds a `lint:package-contents` script that runs the validator across workspaces.
- `.github/workflows/static-checks.yml` executes `npm run lint:package-contents` after type checking.
- Before/after `files` configuration:
  ```json
  // Before
  "files": [ "src", "build-module", "build-types", "*.md" ]
  // After
  "files": [
    "build-module",
    "build-types",
    "src/*-plugins/**",
    "src/prebuilt/css",
    "src/prebuilt/js",
    "*.md",
    "!build-types/**/*.d.mts.map",
    "!build-types/**/*.d.ts.map",
    "!**/stories/**",
    "!**/test/**"
  ]
  ```

## Contribution

Opened and merged by @ciampo with co-authors @jsnajdr, @manzoorwanijk, and @aduth. Initial reviews noted the PR had grown too large with unnecessary Rollup bundling and prepack steps. The author refactored the approach to remove the custom build toolchain, keeping the validation logic lightweight and focused on `npm pack` artifact inspection and export target verification.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
