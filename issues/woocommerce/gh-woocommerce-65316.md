# #65316: Remove `@woocommerce/settings-editor`

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @gigitux
- **Labels:** `needs: documentation`, `plugin: woocommerce`, `focus: monorepo infrastructure`, `package: dependency-extraction-webpack-plugin`
- **Merged:** [`1367ff9`](https://github.com/woocommerce/woocommerce/commit/1367ff979d0225952b9025be9c820d4adc49f8a5)
- **Discussion:** [#65316](https://github.com/woocommerce/woocommerce/pull/65316) · 5 comments · 1 reactions

## Summary

WooCommerce removed the unused `@woocommerce/settings-editor` package and stripped its associated feature-flagged admin plumbing. This cleanup eliminates dead modern-settings frontend sources, REST controllers, transformer classes, and legacy build configurations to reduce monorepo maintenance overhead. The change consolidates the admin settings experience back to the established legacy routing and asset pipeline without altering merchant-facing behavior.

## Impact

- **Extension & Plugin Developers**: No action required unless your code explicitly referenced the removed `@woocommerce/settings-editor` NPM package or shimmed the now-deleted PHP feature classes. Existing extension hooks remain unaffected.
- **Monorepo & Build Infrastructure**: The `@woocommerce/settings-editor` package is fully removed from the bundle manifest and dependency tree. Bundles or local dev environments that still resolve it will fail at build time.
- **Admin Panel / Merchants**: No visible changes. The legacy settings pipeline handles rendering as before, and no configuration migration is needed.

## Technical details

The PR deletes `packages/js/settings-editor/*` along with its Jest, ESLint, Babel, and TypeScript configs, then updates `client/admin/package.json`, `tsconfig.json`, and `webpack.config.js` to drop the module from build targets, asset manifests, and dependency-extraction plugin rules.

On the PHP side, it removes:
- `src/Admin/API/Settings.php` (REST controller)
- `src/Admin/Features/Settings/Init.php` and `Transformer.php` (feature-flagged backend logic)
- Admin bootstrap files (`client/admin/client/settings/index.js`, `settings.scss`) that previously mounted the modern React app.

It also strips `settings` feature flag registrations from `config/core.json` and `config/development.json`, deletes beta-features tracking modal code in `wp-admin-scripts/beta-features-tracking-modal/*`, and simplifies asset enqueuing in `Internal/Admin/WCAdminAssets.php` and `Loader.php` by removing the conditional logic that gated modern settings page rendering. The `WC_Settings_Page` class loses its custom view-handling methods, falling back to standard legacy output. PHPStan baseline suppressions for the deleted classes are cleared in `phpstan-baseline.neon`.

## Contribution

Opened and merged by @gigitux (commit `1367ff9`) as a follow-up to WooCommerce issue #65384. The pull request was labeled `needs: documentation` and `focus: monorepo infrastructure`, indicating its scope as a codebase consolidation effort rather than a user-facing feature or critical bug fix. Review focused on verifying E2E test continuity and confirming the removal of all residual build/PHP references. No alternative architectural approaches were debated; the merge represents a straightforward cleanup to eliminate dead modern-settings plumbing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
