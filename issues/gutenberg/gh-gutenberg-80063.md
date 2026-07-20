# #80063: Theme: Make @wordpress/theme ESM only

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Edit Site`, `[Package] Theme`, `[Package] UI`
- **Merged:** [`7d43785`](https://github.com/WordPress/gutenberg/commit/7d4378585bc1036837aab9622d99f9867ff5c538)
- **Discussion:** [#80063](https://github.com/WordPress/gutenberg/pull/80063) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/theme` package is now ESM-only, removing all published CommonJS JavaScript entrypoints to explicitly define its module contract. This change raises the required Node.js version to `^20.19.0` or `>=22.13.0` and updates the `engines` metadata for `@wordpress/ui` and `@wordpress/edit-site`. The public CSS subpath export remains unchanged, but JavaScript consumers must now use ESM `import` syntax or configure their tooling to handle ESM-only dependencies.

## Impact

- **Plugin & theme developers:** Code using `require('@wordpress/theme')` or relying on dual CJS/ESM exports will fail. Switch to `import` statements or update your bundler configuration.
- **Test runners & tooling:** Jest environments consuming `@wordpress/theme` (or its dependents `@wordpress/ui` and `@wordpress/edit-site`) will need to enable `--experimental-vm-modules` or adjust `transformIgnorePatterns` to process the package from `node_modules`.
- **Hosting & platform teams:** Environments running Node.js `< 20.19.0` (or outside the `>=22.13.0` range) will fail to install or execute packages depending on `@wordpress/theme`. Update your runtime or use a version manager.
- **No action required** if you only import the CSS subpath (`@wordpress/theme/design-tokens.css`) and do not interact with the JavaScript API.

## Technical details

The diff modifies `packages/theme/package.json` to set `"type": "module"`, removes the `build` directory from the `files` array, and strips the `"default"` fallback from the `"exports"` map. The package no longer ships a `build/index.cjs` entrypoint or CJS fallbacks for its subpaths.

Before:
```json
"exports": {
  ".": {
    "types": "./build-types/index.d.ts",
    "import": "./build-module/index.mjs",
    "default": "./build/index.cjs"
  }
}
```

After:
```json
"exports": {
  ".": {
    "types": "./build-types/index.d.ts",
    "default": "./build-module/index.mjs"
  }
}
```

The `@wordpress/design-tokens.js` subpath now points directly to `./src/prebuilt/js/design-tokens.mjs` to allow validation and linting before the build output exists, while runtime entrypoints continue to target `build-module`. The `prelint:js` script that previously generated a CJS fallback for `design-tokens.js` was removed from the `build` script. The `engines.node` field is updated across `@wordpress/theme`, `@wordpress/ui`, and `@wordpress/edit-site` to `^20.19.0 || >=22.13.0`, and `npm` to `>=10.2.3`.

## Contribution

Opened by `@ciampo` and merged in commit `7d43785`, this PR was driven by the Gutenberg package stabilization checklist to remove remaining CommonJS entrypoints and lock the JavaScript module contract. During review, `@jsnajdr` and `@anomiex` raised concerns about Jest compatibility, noting that older test suites might struggle with ESM-only exports. The team confirmed that Jest v30+ supports `require(ESM)` via `--experimental-vm-modules` or Babel transformation workarounds, and agreed this would not block the merge. `@manzoorwanijk` and `@aduth` weighed in on the Jest ecosystem status, confirming the workaround path before the PR was approved.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
