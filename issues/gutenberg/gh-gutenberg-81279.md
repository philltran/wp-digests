# #81279: Address GHSAs for phpcs, wpcs

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @sirreal
- **Labels:** `[Type] Security`
- **Merged:** [`23b5b85`](https://github.com/WordPress/gutenberg/commit/23b5b854ed0b36853fc4d8624368224c839d6bdd)
- **Discussion:** [#81279](https://github.com/WordPress/gutenberg/pull/81279) · 5 comments · 0 reactions
- **Usefulness:** 2/5

## Summary

Gutenberg's Composer dev dependencies are bumped to pick up fixes for two GitHub Security Advisories, one in PHP_CodeSniffer and one in WordPress-Coding-Standards. `wp-coding-standards/wpcs` moves to `^3.4.1`, and `squizlabs/php_codesniffer` is added as an explicit `require-dev` entry at `^3.13.6` so a patched phpcs is guaranteed rather than left to transitive resolution.

## Impact

- **Gutenberg contributors / forks:** run `composer update` to pick up the new lock resolution. Linting behavior may shift slightly with newer phpcs/wpcs releases, so new sniff results are possible.
- **Plugin & theme developers using Gutenberg's PHP tooling:** no runtime effect. These are dev-only dependencies and do not ship in the plugin.
- **Other projects:** projects that depend on `wpcs` or `phpcs` in CI or local tooling may want to check their own constraints against the two advisories. The PR does not describe the vulnerabilities beyond linking them.
- **Site owners / hosting / REST consumers:** no action required.

## Technical details

The diff touches two `composer.json` files only:

- Root `composer.json` (`require-dev`): `wp-coding-standards/wpcs` `^3.0` → `^3.4.1`, and a new `squizlabs/php_codesniffer: ^3.13.6` entry.
- `test/php/gutenberg-coding-standards/composer.json`: `squizlabs/php_codesniffer` `^3.7.2` → `^3.13.6` in `require`, and `wp-coding-standards/wpcs` `^3.0` → `^3.4.1` in `require-dev`.

The explicit phpcs constraint acts as a floor. Previously wpcs's own requirement could have allowed an older phpcs to resolve. The author's check on the branch showed `php_codesniffer` 3.13.6 and `wpcs` 3.4.1 installed. No source code, hooks, or PHP runtime behavavior changed.

```json
"wp-coding-standards/wpcs": "^3.4.1",
"squizlabs/php_codesniffer": "^3.13.6"
```

## Contribution

The author (@sirreal) added backport labels so the dependency update lands on any branches still in use, and @ciampo cherry-picked it to `release/23.7` so it ships in the next release. CI reported only an unrelated flaky e2e test.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
