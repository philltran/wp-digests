# #79394: Boot: run page `init` modules in `initSinglePage`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] wp-build`, `[Feature] Dashboard`
- **Merged:** [`31a7652`](https://github.com/WordPress/gutenberg/commit/31a7652d8fd9f038dd3aac6d5c7f75c94eb2dc30)
- **Discussion:** [#79394](https://github.com/WordPress/gutenberg/pull/79394) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/boot` package now executes page `init` modules during the `initSinglePage()` boot path, matching the behavior of `init()`. Previously, embedded wp-admin pages silently ignored `init` modules, forcing developers to register startup logic in route lifecycle hooks instead. This change introduces a shared `runInitModules()` helper and updates the build template to forward declared init modules to the boot function, enabling reliable page-level startup hooks for embedded applications.

## Impact

- **Plugin & theme developers:** No action required for existing code. Developers who previously relied on route `beforeLoad` hooks to register entities or configure menus for embedded wp-admin pages should migrate that logic to `init` modules for consistency.
- **Gutenberg/Core contributors:** The dashboard page configuration now requires the `@wordpress/dashboard-init` package in its `pages[].init` array. The legacy `routes/dashboard/route.ts` file is removed.
- **Hosting & platform teams:** No configuration changes needed; the PHP template automatically generates `$init_modules` arrays when the build config declares them.

## Technical details

The diff refactors `packages/boot/src/components/app/index.tsx` to extract the init loop into an awaited `runInitModules( initModules )` helper. Both `init()` and `initSinglePage()` now accept an `initModules?: string[]` parameter and execute it before rendering. The `page-wp-admin.php.template` in `packages/wp-build/templates/` is updated to inject `$init_modules = {{INIT_MODULES_JSON}}` and pass it to the inline boot call:

```diff
- mod.initSinglePage( { mountId, routes } );
+ mod.initSinglePage( { mountId, routes, initModules } );
```

The dashboard's `widgetModule` entity registration moves from `routes/dashboard/route.ts` (which previously used a `beforeLoad` hook) to a new `@wordpress/dashboard-init` package that exports an `async function init()`. The build config in `package.json` declares this via `"init": ["@wordpress/dashboard-init"]` under the dashboard page. Documentation for the boot package and the new dashboard-init package was added to align with monorepo conventions.

## Contribution

Opened by @retrofox to address a silent no-op in embedded page boots, the PR was reviewed and merged with contributions from @chihsuan and @youknowriad. The author initially considered keeping the route lifecycle hook but recognized that `init` modules provided a symmetric, once-per-page startup primitive. Documentation for the boot package and the new dashboard-init package was added during review to align with existing monorepo conventions.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
