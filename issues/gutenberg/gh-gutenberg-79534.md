# #79534: Divider: Restore lower border specificity

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Components`
- **Merged:** [`9da5358`](https://github.com/WordPress/gutenberg/commit/9da5358d373dc8578d2951981b2595f8b7a9095c)
- **Discussion:** [#79534](https://github.com/WordPress/gutenberg/pull/79534) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Resolves a CSS specificity regression in the `Divider` component caused by a recent SCSS module migration. A prior update moved default border declarations into attribute selectors, inadvertently increasing their cascade weight and preventing legacy theme overrides from applying. This patch wraps the orientation-specific blocks in a `:where()` pseudo-class to restore lower specificity, ensuring custom border rules continue to take precedence.

## Impact

- **Theme & plugin developers:** Themers relying on complementary areas, global styles, or wrapper classes to override `.divider` borders will see their custom shorthands apply correctly again.
- **Block library consumers:** No immediate code changes required. Verify post-update behavior in components that render the `Divider` block.
- **Action required:** None for standard installations; themers should audit border overrides if they relied on the previously broken cascade.

## Technical details

The change lives in `packages/components/src/divider/style.module.scss`. The diff replaces raw attribute selectors with `&:where([aria-orientation="horizontal"])` and `&:where([aria-orientation="vertical"])`.

**Before/After pattern:**
```scss
/* Before */
[aria-orientation="horizontal"] { border-block-end: 1px solid currentColor; block-size: 0; ... }

/* After */
&:where([aria-orientation="horizontal"]) { border-block-end: 1px solid currentColor; block-size: 0; ... }
```
Wrapping the attribute selectors in `:where()` zeroes out their contribution to CSS specificity calculations. This drops them back below standard class-level ranks, ensuring that external theme wrappers or complementary-area rules setting full `border` shorthands outrank the component's internal defaults.

## Contribution

Opened by @ciampo as a follow-up fix to #79444. During review, @t-hamano suggested applying `:where()` only to the vertical orientation for cleaner diffs and targeted specificity alignment. The author opted to wrap both orientations instead to maintain consistent baseline behavior across layouts. Merged in commit `9da5358` with a negligible size increase (+4 B gzipped).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
