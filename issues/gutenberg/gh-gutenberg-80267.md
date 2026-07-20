# #80267: Theme: Widen peer dependency ranges to support new versions of Vite, ESBuild, and Stylelint

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] Enhancement`, `[Package] Theme`
- **Merged:** [`efb2907`](https://github.com/WordPress/gutenberg/commit/efb2907a65c3c4a36f2ac3bf33bd22774e7b236c)
- **Discussion:** [#80267](https://github.com/WordPress/gutenberg/pull/80267) · 4 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/theme` package has widened its optional peer dependency ranges to support newer major versions of Vite, esbuild, and Stylelint. This prevents npm and yarn peer resolution conflicts when downstream projects use `@wordpress/theme` alongside modern build tooling. It matters because consumers previously encountered installation errors when using Vite 8, Stylelint 17, or esbuild 0.28+ even if they never intended to use the theme's Vite plugin.

## Impact

- **Plugin & theme developers / build tooling users:** No code changes required. Projects using `@wordpress/theme` with Vite 8, Stylelint 17, or esbuild 0.28+ will now install without peer dependency warnings or errors.
- **Hosting & platform teams:** No configuration changes needed. The package remains backward-compatible with previously supported versions.
- **Maintenance note:** The `@wordpress/theme` maintainers will need to manually audit and update these ranges when esbuild, Stylelint, or Vite release new major versions, as the ranges are no longer pinned to a single major line.

## Technical details

The change modifies the `peerDependencies` field in `packages/theme/package.json` (and the corresponding `package-lock.json`):
- `esbuild`: `^0.27.2` → `>=0.27.2 <1.0.0`
- `stylelint`: `^16.8.2` → `^16 || ^17`
- `vite`: `^7.3.2` → `^7 || ^8`
These are optional peer dependencies, so validation only triggers if the consuming project also lists them as dependencies. The author confirmed compatibility with the newer versions before widening the ranges, noting that Vite's peer validation still applies even when the `@wordpress/theme` Vite plugin is not actively used.

## Contribution

Opened and merged by @aduth, with co-authors @ciampo, @anomiex, and @simison. The PR initially targeted only Vite but was expanded during review to cover esbuild and Stylelint after verifying compatibility across all three. The author noted that maintaining these ranges will require ongoing updates for future major releases, as the team opted for explicit version ranges rather than open-ended `>=` or `*` constraints to avoid accepting future incompatibilities.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
