# #78882: Packages: Declare missing `@types/react` dependency

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @manzoorwanijk
- **Labels:** `[Type] Code Quality`, `[Package] Data`, `[Package] Server Side Render`, `[Package] Compose`, `[Package] Core data`, `[Package] Viewport`, `[Package] Keycodes`, `[Package] Plugins`, `[Package] Components`, `[Package] Blocks`, `[Package] Editor`, `[Package] Block library`, `[Package] Notices`, `[Package] Rich text`, `[Package] Block editor`, `[Package] Primitives`, `[Package] Icons`, `[Package] Keyboard Shortcuts`, `[Package] React i18n`, `[Package] Style Engine`, `[Package] Preferences`, `[Package] Media Utils`, `[Package] DataViews`, `[Package] Fields`, `[Package] Router`, `[Package] Theme`, `[Package] UI`
- **Merged:** [`bf0b188`](https://github.com/WordPress/gutenberg/commit/bf0b188aa0fb8feb7f4c0010ef93b14d7dc2c742)
- **Discussion:** [#78882](https://github.com/WordPress/gutenberg/pull/78882) · 10 comments · 0 reactions

## Summary

Adds explicit `@types/react` and `@types/react-dom` declarations to multiple WordPress/Gutenberg packages that previously relied on ambient resolution from root-level hoisting. This prevents TypeScript from silently falling back to `any` when `skipLibCheck: true` is enabled and fixes broken type resolution in isolated package manager layouts (e.g., pnpm).

## Impact

['**Plugin, Theme & Package Developers:** No runtime or behavioral changes. Requires a fresh dependency install (`npm install`, `pnpm install`) to resolve the newly declared type packages correctly during local builds.', '**Headless & Block Editor Consumers:** Improves reliability of TypeScript type checking when consuming published block editor packages from npm registries, eliminating silent type narrowing to `any`.']

## Technical details

Modifies `package.json` files across the `packages/` ecosystem. The change applies a deterministic rule: if a package's emitted `build-types/*.d.ts` files contain inline React type imports (e.g., `import('react').ReactNode`), `@types/react` is added to its `dependencies`. If React types are only consumed internally and not exported in the declaration output, they are added to `devDependencies`. `@types/react-dom` follows the same emission-based logic. No JavaScript/TypeScript source files, build scripts, or configuration values were altered; the diff strictly updates dependency maps and corresponding changelog entries.

## Contribution

Opened by @manzoorwanijk as part of an ongoing dependency hygiene sweep (related to #75041 and #74310). Co-authored and reviewed by @aduth and @jsnajdr. The PR was accepted immediately after automated scanning of `build-types/` output confirmed the missing declarations, with no alternative approaches debated due to the standardized nature of npm workspace dependency resolution.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
