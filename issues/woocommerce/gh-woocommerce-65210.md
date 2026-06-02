# #65210: Replace `tsc` With `esbuild`

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @ObliviousHarmony
- **Labels:** `focus: monorepo infrastructure`, `package: @woocommerce/components`, `package: @woocommerce/csv-export`, `package: @woocommerce/currency`, `package: @woocommerce/customer-effort-score`, `package: @woocommerce/data`, `package: @woocommerce/date`, `package: dependency-extraction-webpack-plugin`, `package: @woocommerce/eslint-plugin`, `package: @woocommerce/experimental`, `package: @woocommerce/explat`, `package: @woocommerce/navigation`, `package: @woocommerce/number`, `package: @woocommerce/onboarding`, `package: @woocommerce/tracks`, `package: @woocommerce/e2e-utils-playwright`
- **Merged:** [`7d6e8c4`](https://github.com/woocommerce/woocommerce/commit/7d6e8c41e5aa9b4663c29580b59e089ab9f8010a)
- **Discussion:** [#65210](https://github.com/woocommerce/woocommerce/pull/65210) · 9 comments · 0 reactions

## Summary

This monorepo infrastructure change swaps the TypeScript compiler (`tsc`) for `esbuild` across all per-package JS builds in the WooCommerce monorepo, building on a prior PR (#65168) that had already stripped type-checking out of the build pipeline. Because type-checking no longer ran during builds, `tsc` was only doing transpilation — work `esbuild` does dramatically faster. The result is large build-time and memory reductions: cold builds drop ~71% (44.7s → 12.8s) and watch-mode peak memory falls ~68% (24.4 GB → 7.9 GB). As a side effect, JSDoc comments are now stripped from transpiled output, and the TS config switches to `isolatedModules`, which requires per-file-safe code (e.g. explicit `import type`).

## Impact

**Plugin & extension developers (consumers of `@woocommerce/*` JS packages)**
- The published/transpiled `.js` in affected packages (`@woocommerce/components`, `data`, `currency`, `navigation`, `number`, `date`, `onboarding`, `tracks`, `experimental`, `explat`, `customer-effort-score`, `csv-export`, `product-editor`, etc.) no longer contains JSDoc comment blocks. Generated `.d.ts` type declarations **retain** their documentation, so IDE/typed tooling is unaffected; only the runtime JS lost inline comments.
- Per-file transpiled bundle sizes grew (overall +186 kB / +3.27%), largely from per-file boilerplate and the appearance of standalone `types.js` files — a structural artifact of `isolatedModules`, not new runtime logic.
- No public API, hook, or REST change. Runtime behavior of the packages is unchanged.

**Contributors to the WooCommerce monorepo**
- TypeScript config now uses `isolatedModules`. New/modified code must be single-file-transpile-safe: type-only imports must use `import type { Foo } from 'bar'` (or inline `import { type Foo }`), or transpilation produces invalid JS.
- Google Wireit was removed from per-package builds (its overhead no longer pays off given esbuild's speed); it remains only for bundle builds for now.
- Watch-mode steady-state CPU rose (~3% → ~31%) because the model spawns ~25 per-package esbuild daemons each doing keepalive work — a known trade-off, expected to resolve if per-package builds later collapse into a shared esbuild process.

**Site owners / hosting / headless & REST consumers**
- No action required. This is a build-tooling change with no server-side, database, or REST surface impact.

## Technical details

The change removes `tsc`-based compilation from per-package build scripts and replaces it with `esbuild` (invoked via per-package `build.mjs` esbuild wrapper scripts), mirroring how `@wordpress/*` packages build. Key mechanics:

- **`isolatedModules` enabled** in the TypeScript configuration. Because `esbuild` operates per-file with no cross-file type resolution, a re-exported type imported without the `type` modifier would be emitted as a real (undefined) runtime import. Code must therefore distinguish value vs. type imports:

```ts
// Before (worked under tsc, which resolved Foo as a type and erased it)
import { Foo } from 'bar';

// After (required under isolatedModules + esbuild)
import type { Foo } from 'bar';
```

- **JSDoc stripped from output.** `esbuild`'s transform pass drops JSDoc comment blocks from the transpiled `.js`. The generated `.d.ts` declarations still carry documentation.
- **Wireit removed from package builds.** `google/wireit` orchestration was dropped from per-package builds; it is retained for bundle builds.
- **Process model in watch mode** (`pnpm watch:build`): the 24 `tsc --watch` processes (~6.5 GB) are replaced by ~25 `node build.mjs` wrappers (~1.5 GB) plus ~25 `esbuild --service` daemons (~1.0 GB), a net ~4 GB reduction; webpack (~4.5 GB) becomes the dominant memory consumer. Time-to-settle drops from 132s to ~30s.

The build-size report shows new per-directory `types.js` files (e.g. `build/.../types.js` at ~368 B each) appearing across `@woocommerce/product-editor` — a consequence of per-file emission under the new model rather than added functionality.

## Contribution

Authored by @ObliviousHarmony and merged as commit `7d6e8c4`, the PR is the second step of a two-part effort: it directly builds on #65168 (which removed type-checking from the build pipeline, making `tsc` redundant) and closes #64838. The discussion thread is largely automated tooling output (Playground preview, bundle-size report, testing-guidelines reminder) rather than design debate; reviewers were asked to lean on CI as the primary signal and to watch CI performance given the Wireit removal. The author explicitly flagged follow-up work: revisiting JSDoc preservation, and collapsing the ~25 per-package esbuild daemons into a shared process to address the raised steady-state CPU.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
