# #82221: Private APIs: Add back '@wordpress/dataviews' to the allowed core modules

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `Backwards Compatibility`, `[Package] Private APIs`
- **Merged:** [`45b200e`](https://github.com/WordPress/gutenberg/commit/45b200e9b8a6a2c816ae0c06bb60ebfab46ded33)
- **Discussion:** [#82221](https://github.com/WordPress/gutenberg/pull/82221) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Gutenberg 23.9.0-rc.1 removed `@wordpress/dataviews` from the `CORE_MODULES_USING_PRIVATE_APIS` allowlist in `@wordpress/private-apis`. Copies of `@wordpress/dataviews` published to npm before that cleanup call the private-APIs opt-in at module load, so plugin bundles embedding them threw "You tried to opt-in to unstable APIs" and broke. This PR restores the allowlist entry, and it was cherry-picked into the 23.9 release branch.

## Impact

- **Plugin developers bundling `@wordpress/dataviews`:** Bundles containing an older npm copy (Jetpack Publicize and the WordPress AI plugin are named as examples) no longer throw at load time once this fix ships. No code change is needed for the opt-in error itself.
- **Site owners:** On 23.9.0-rc.1, affected plugins failed in the post editor. For example, the Jetpack sidebar was missing and the console showed the opt-in error. This fix restores that behavior.
- **Remaining risk:** A reviewer noted that older bundled copies may also look up specific private APIs that were removed in newer versions (the removals from #81230). The allowlist entry only prevents the `unlock`/opt-in error, so those copies may still break. Mitigations mentioned are moving to the `/wp` entrypoint or updating to a newer DataViews release, both of which need downstream action.
- **Gutenberg maintainers:** The PR says no strategy exists yet for when such allowlist entries can be removed.

## Technical details

The change adds `'@wordpress/dataviews'` back to the `CORE_MODULES_USING_PRIVATE_APIS` array in `packages/private-apis/src/implementation.ts`, right after `'@wordpress/theme'` and before `'@wordpress/fields'`. It carries a "Do not remove" comment explaining that older npm-published DataViews versions call the opt-in at module load, so a plugin bundling one of those copies throws if the entry is missing.

The private-apis opt-in only accepts module names on this allowlist, and any other name triggers the "You tried to opt-in to unstable APIs" error. The PR also adds a bug-fix entry to `packages/private-apis/CHANGELOG.md`. The built `build/scripts/private-apis/index.min.js` grows by about 2 B.

## Contribution

This is a follow-up to #81478, which removed the entry. It was merged and then cherry-picked by @oandregal into `release/23.9` so it lands in the 23.9 release. In review, @aduth questioned whether the allowlist fix is enough, given that private APIs removed in #81230 may still be referenced by older bundled copies. @ciampo agreed this is possible and pointed to a comment on #81230 about a potential compat feature. The PR notes AI tooling was used to generate the change, with manual review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
