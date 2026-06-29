# #79077: Add React 19 as an experimental flag

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jsnajdr
- **Labels:** `[Type] Enhancement`, `[Package] Element`, `[Package] Block editor`, `Backport to Gutenberg RC`, `No Core Sync Required`
- **Merged:** [`8bc8489`](https://github.com/WordPress/gutenberg/commit/8bc848998e0d2ff2f59634363dd03a0c92212ffc)
- **Discussion:** [#79077](https://github.com/WordPress/gutenberg/pull/79077) · 17 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Introduces the `gutenberg-react-19` experimental flag to conditionally load React 19 as the bundled vendor runtime alongside the existing React 18 build. The change includes polyfills that simulate React 18's JSX behavior (e.g., `defaultProps` resolution, legacy `Symbol.for('react.element')` tagging) and addresses altered portal rendering in React 19. It also removes deprecated `findDOMNode`-related wrappers from the `@wordpress/element` package.

## Impact

- **Plugin & theme developers:** No immediate migration required for standard usage. Enable the experiment flag to surface early compatibility warnings or runtime errors related to React 19 API changes (implicit `defaultProps`, `inert` attribute typing, and portal rendering).
- **Hosting & platform teams:** The flag provides a safe beta testing channel without forcing production sites off stable React 18. Requires awareness of the short-lived rollout window discussed during review.
- **Core/Block developers:** Deprecated polyfill wrappers (`findDOMNode`, `render`, `hydrate`, `unmountComponentAtNode`) are stripped from `@wordpress/element`; use DOM refs and `createRoot`/`hydrateRoot` instead.

## Technical details

- Registers the `gutenberg-react-19` experiment in `lib/experimental/experiments/load.php` via `gutenberg_initialize_experiments_settings()`.
- When enabled, `gutenberg_register_vendor_scripts()` in `lib/client-assets.php` resolves the `react`, `react-dom`, and `react-jsx-runtime` handles to `-19` suffixed assets (e.g., `build/scripts/vendors/react-19.min.js`) and their `.asset.php` manifests.
- Adds a version guard in `packages/block-editor/src/components/iframe/index.js` (`reactVersion.split('.')[0] === '18'`) that conditionally injects a self-removing script tag into the `<body>`, addressing differences in how React 19 handles portal rendering.
- Ships internal compat shims (polyfilling `ReactCurrentOwner` and `ReactCurrentDispatcher`) so plugins using bundled React 18 JSX runtimes produce elements tagged with `Symbol.for('react.element')`.
- Removes `packages/element/src/react-polyfill.ts` and its tests, stripping legacy `findDOMNode` polyfills from the package export path while leaving modern block-editor compatibility intact.
- Adds an e2e smoke test plugin (`packages/e2e-tests/plugins/react-18-compat-block`) to verify mixed-runtime interoperability.

## Contribution

Merged by @jsnajdr with co-authors @manzoorwanijk, @tyxla, @aduth, and @youknowriad. The PR was structured as a direct follow-up to the reverted React 19 upgrade (#61521/#78940), implementing a dual-runtime strategy via an experiment flag rather than forcing an immediate default switch. Reviewers emphasized a short-lived rollout window to push plugin authors toward migration, but the final merged diff landed strictly with the experiment registration, compat polyfills, and e2e coverage without altering production defaults.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
