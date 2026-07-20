# #80422: Fix Root `.prettierrc.js` override issue

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shail-mehta
- **Labels:** `[Type] Bug`, `[Package] Base styles`, `[Package] Theme`
- **Merged:** [`80297f8`](https://github.com/WordPress/gutenberg/commit/80297f8e6aa4b968a03b59603d4962253b4a341a)
- **Discussion:** [#80422](https://github.com/WordPress/gutenberg/pull/80422) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg repository renamed its root Prettier configuration from `.prettierrc.js` to `prettier.config.mjs` and fixed a bug where the root config was overwriting shared CSS/SCSS formatting overrides from `@wordpress/prettier-config`. This ensures that CSS, SCSS, and `changelog.txt` files consistently receive the correct parser and quote settings across editor integrations and the `wp-scripts format` command.

## Impact

- **Gutenberg core contributors & plugin/theme developers working in the monorepo:** Must update local editor configurations, IDE settings, and CI scripts that explicitly reference `.prettierrc.js` to use `prettier.config.mjs`.
- **Build & CI pipelines:** Custom scripts or GitHub Actions that resolve or lint against the old config path will need updating.
- **No action required** for end-users or standard WordPress site owners, as this is strictly a development tooling change within the Gutenberg repository.

## Technical details

The root configuration file was renamed from `.prettierrc.js` to `prettier.config.mjs`. Previously, the config spread `@wordpress/prettier-config` and then replaced its `overrides` array, which dropped the shared `singleQuote: false` and `parenSpacing: false` rules for CSS/SCSS. The new config appends the `changelog.txt` override to the shared overrides array instead of replacing it. `packages/scripts/utils/config.js` was updated so `wp-scripts format` detects `prettier.config.mjs` rather than injecting a bundled fallback. The change also regenerated `packages/base-styles/internal/_wpds-token-fallbacks.scss` and `packages/theme/prebuilt/css/design-tokens.css` to conform to the corrected double-quote formatting.

## Contribution

Opened and merged by @shail-mehta, with review contributions from @ciampo. The PR closed #80332 and superseded #80365. Discussion was limited to verifying config resolution and updating contributor documentation, with no alternative approaches debated in the provided comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
