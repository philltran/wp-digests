# #79024: Media Editor: Remove lag when toggling the sidebar

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Media`
- **Merged:** [`57f0ab2`](https://github.com/WordPress/gutenberg/commit/57f0ab2f9eb5729a71cfcffcdbd06d7fbba2944b)
- **Discussion:** [#79024](https://github.com/WordPress/gutenberg/pull/79024) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a UI regression in the Gutenberg media editor where toggling the details/crop sidebar caused a noticeable 300ms lag instead of animating smoothly. The issue was introduced when a CSS rule pinned the sidebar width with `!important` to fix overflow at certain breakpoints, which inadvertently froze the inline width animation driven by `ComplementaryAreaFill`. The fix restricts that width override to only apply between the small and medium breakpoints, restoring the expected open/close animation above 782px.

## Impact

- **Site owners & editors:** Restores smooth, immediate sidebar toggling in the media editor modal without visual stutter.
- **Plugin & theme developers:** No direct API or code changes required. If you are extending the media editor UI or overriding `InterfaceSkeleton`/`ComplementaryArea` styles, note that the `!important` width rule is now scoped to `600px–781px` viewports.
- **Hosting & platform teams:** No action required. This is a client-side CSS adjustment in the Gutenberg package.

## Technical details

The change modifies `packages/media-editor/src/components/media-editor/style.scss`. Previously, a rule targeting `.interface-interface-skeleton__sidebar:has(.interface-complementary-area__fill)` applied `position: relative !important;` unconditionally at `break-small`. This clobbered the inline `width` style that `ComplementaryAreaFill` tweens over 300ms, causing the panel to remain static until `AnimatePresence` unmounted it. The diff replaces the unconditional breakpoint rule with a bounded media query:

```scss
@media (min-width: #{ breakpoints.$break-small }) and (max-width: #{ (breakpoints.$break-medium - 1) }) {
	.interface-interface-skeleton__sidebar:has(.interface-complementary-area__fill) {
		position: relative !important;
	}
}
```

This ensures the `!important` positioning only applies in mobile/inline mode (where `ComplementaryArea` already zeroes the animation duration), while leaving the desktop layout to `InterfaceSkeleton` and allowing the width tween to animate freely above 782px.

## Contribution

Opened and merged by @ramonjd as a follow-up to #78931, with co-authorship from @andrewserong. The PR was reviewed quickly, with @talldan noting the original observation and @andrewserong confirming the fix. During review, @ramonjd cleaned up an outdated explanation in the PR description that referenced a discarded CSS approach, leaving only the final scoped media query change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
