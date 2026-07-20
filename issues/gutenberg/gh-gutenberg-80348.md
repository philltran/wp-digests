# #80348: Fix: Critical error websocket-driver fix

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @im3dabasia
- **Labels:** `npm Packages`, `[Type] Security`
- **Merged:** [`a3d8763`](https://github.com/WordPress/gutenberg/commit/a3d8763d7901ae03665c2dd459be8f85a4e11743)
- **Discussion:** [#80348](https://github.com/WordPress/gutenberg/pull/80348) · 1 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

A critical security vulnerability in the `websocket-driver` npm package was patched by upgrading the dependency from `0.7.4` to `0.7.5` in the Gutenberg monorepo. This resolves a Dependabot security alert and mitigates potential exploitation risks in environments that execute Gutenberg's Node-based tooling or build pipelines.

## Impact

- **Plugin & theme developers / platform teams:** No code changes required. Run `npm install` or update your `package-lock.json` to pull the patched `0.7.5` version.
- **CI/CD & hosting environments:** Ensure build steps that install Gutenberg dependencies are updated to avoid legacy lockfiles retaining the vulnerable `0.7.4` release.
- **No breaking changes, deprecations, or API modifications.**

## Technical details

The unified diff modifies only `package-lock.json`. It updates the `node_modules/websocket-driver` entry, bumping the `version` from `0.7.4` to `0.7.5`, replacing the `integrity` hash, and adding a `license` field (`Apache-2.0`). The change was applied via `npm audit fix` and does not touch any PHP, JavaScript, or block registration code.

## Contribution

Opened by @im3dabasia to close Dependabot alert #702. Merged with co-authorship from @manzoorwanijk. The approach was straightforward: running `npm audit fix` to resolve the critical vulnerability, with no alternative implementations or design debates required.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
