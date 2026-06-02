# #78804: Icons: rename timeToRead to time

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Package] Icons`
- **Merged:** [`a1545c5`](https://github.com/WordPress/gutenberg/commit/a1545c57c1586a7a93dda92c3d8dee99e28ef166)
- **Discussion:** [#78804](https://github.com/WordPress/gutenberg/pull/78804) · 11 comments · 0 reactions

## Summary

The `timeToRead` icon in `@wordpress/icons` is renamed to `time`. The icon — a clock/hourglass glyph — was introduced in PR #72240 alongside the Post Time to Read block, but its feature-specific name discouraged reuse in unrelated contexts. Renaming it to `time` makes it a general-purpose icon, consistent with how other broadly applicable icons in the library are named.

## Impact

**Plugin and theme developers**
- Any code importing `timeToRead` by name from `@wordpress/icons` may need to be updated to `time`.
- The PR description states the icons package is bundled via `wp-scripts`/`wp-build`, and implies the old export name may not be retained — confirm against your Gutenberg version before upgrading.
- The visually similar `scheduled` icon is unaffected and remains under its existing name.

**No action required** for users of the Post Time to Read block — the block's internal icon reference is a package-level implementation detail.

## Technical details

The change is in the `@wordpress/icons` package (`packages/icons/src/`). The named export `timeToRead` is replaced by `time`. The PR description notes: *"The icons package is bundled when building with wp-scripts/wp-build, so we don't need to worry about breaking changes and keep exporting old name"* — this statement is ambiguous as to whether a backward-compat re-export of `timeToRead` is preserved alongside `time`, or whether the old name is simply dropped. The reported build size delta of **0 B** across all bundles is consistent with either a pure rename or a rename plus a trivial alias. Without the diff, the exact treatment of the old identifier cannot be confirmed.

Before/after usage pattern if no alias is kept:
```js
// Before
import { timeToRead } from '@wordpress/icons';

// After
import { time } from '@wordpress/icons';
```

Note that a separate `scheduled` icon exists with a similar clock appearance; this PR does not merge or alias those two icons.

## Contribution

Opened and merged by @simison (commit `a1545c5`). Discussion explored three alternatives: keeping the name `timeToRead`, using `clock` (rejected as less generic than `time`), and renaming the existing `scheduled` icon to `time` while leaving `timeToRead` unchanged. The author and reviewers settled on `time` as the most generic, convention-consistent choice. AI tooling was disclosed as part of the authoring process per WordPress AI Guidelines.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
