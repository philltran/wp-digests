# #78847: e2e-test-utils-playwright: add src to published NPM files

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jsnajdr
- **Labels:** `[Type] Regression`, `[Package] E2E Tests`
- **Merged:** [`35487a0`](https://github.com/WordPress/gutenberg/commit/35487a041c6569399988a96d5956b92cd3da0a71)
- **Discussion:** [#78847](https://github.com/WordPress/gutenberg/pull/78847) · 7 comments · 0 reactions

## Summary

This patch fixes a regression in the published `@wordpress/e2e-test-utils-playwright` NPM package. In PR #78237, the author stopped building the package — relying on Playwright's ability to consume TypeScript/ESM source files directly — which let e2e test runs skip a build step. However, the `src` directory was never added to the `files` array in the package's `package.json`, so the published tarball shipped without any source files, leaving the package effectively empty for consumers installing it from NPM. The fix adds `src` to `files`.

## Impact

- **Plugin & theme developers / test authors:** Anyone who installs `@wordpress/e2e-test-utils-playwright` from NPM (rather than consuming it from the Gutenberg monorepo) was affected — the package shipped without its source, so imports would fail. After this fix, the published package again contains the consumable files. No code changes are required on the consumer side; updating to the fixed release is sufficient.
- **No action required** for developers working inside the Gutenberg monorepo, where the change worked correctly because source files are present locally. The bug only manifested in the published NPM artifact.
- No API surface, exports, or usage patterns changed — this is purely a packaging fix.

## Technical details

The change targets the `files` field in the package's `package.json`. After #78237 removed the build step for `e2e-test-utils-playwright`, the package is intended to be consumed directly from its `src` directory, but `src` was omitted from the whitelist of files included in the published tarball.

Before (conceptually), `files` did not list the source directory, so `npm publish` excluded it. After this patch, `src` is added:

```jsonc
"files": [
  "src"
]
```

With `src` whitelisted, `npm pack`/`npm publish` now includes the source files that Playwright consumes, restoring the package to a working state. The CI "Size Change" report shows 0 B change to the built bundles, consistent with this being a publishing-manifest fix rather than a code change.

## Contribution

Authored and merged by @jsnajdr as a regression fix (labeled `[Type] Regression`, `[Package] E2E Tests`), merged in commit `35487a0`. It resolves issue #78832, reported by @Menrath, and traces back to a regression introduced by the author's own earlier PR #78237. Props also recorded for @Mamaduka. The discussion was limited to automated props-bot and bundle-size reporting, with no notable design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
