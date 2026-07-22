# #79471: Theme: Add Sass fallback helper root entrypoint

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Base styles`, `[Package] Theme`
- **Merged:** [`fda0164`](https://github.com/WordPress/gutenberg/commit/fda0164e0967405ecd2fd7531c74344bfb67a1d6)
- **Discussion:** [#79471](https://github.com/WordPress/gutenberg/pull/79471) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/theme` package now exposes a stable Sass entry point at `@wordpress/theme/design-token-fallbacks` via a new root-level shim file. This change replaces direct imports into the generated `src/prebuilt` directory, ensuring the fallback helper resolves consistently across modern package-export-aware bundlers and legacy Sass `node_modules` load-path setups. It also corrects a token name typo in the documented example.

## Impact

- Plugin & theme developers importing `@wordpress/theme/design-token-fallbacks`: No action required. The import path remains identical, but it now routes through a stable shim instead of an internal `src/prebuilt` directory.
- Developers copying the README example: The documented token name was corrected from `--wpds-color-fg-content-neutral` to `--wpds-color-foreground-content-neutral`, which may require updating copied code.
- Build tooling: The `package.json` export now uses a `sass` condition, aligning with modern package resolution standards without breaking existing setups.

## Technical details

- Added `packages/theme/design-token-fallbacks.scss` as a root shim containing `@forward "./src/prebuilt/scss/design-token-fallbacks";`.
- Updated `packages/theme/package.json` to map `"./design-token-fallbacks"` to a conditional export with `"sass": "./design-token-fallbacks.scss"`, and added the shim to the `files` array.
- Updated `packages/base-styles/_mixins.scss` to import via the new stable path.
- Removed `'--wpds-dimension-base': '4px',` from the `$fallbacks` map in `packages/theme/src/prebuilt/scss/design-token-fallbacks.scss`.
- Before/after import pattern:
  ```scss
  // Before (internal path)
  @use "@wordpress/theme/src/prebuilt/scss/design-token-fallbacks" as wpds;
  // After (stable public path)
  @use "@wordpress/theme/design-token-fallbacks" as wpds;
  ```

## Contribution

Opened and merged by @mirka, with co-authors @ciampo and @aduth. The PR introduces a root-level Sass shim to stabilize the `@wordpress/theme/design-token-fallbacks` import path, preventing generated `src/prebuilt` directories from leaking into the public API. It also corrects a token name typo in the README example and removes a redundant fallback variable. The change was reviewed and merged with minimal discussion, explicitly unblocking previously reverted Sass resolution logic from #79429.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
