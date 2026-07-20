# #80105: fix: set dataviews popover hover text color

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Pranjal1423
- **Labels:** `[Type] Bug`, `First-time Contributor`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Package] DataViews`
- **Merged:** [`3ca4372`](https://github.com/WordPress/gutenberg/commit/3ca437274420787ebf6a423308e98a10e91f548d)
- **Discussion:** [#80105](https://github.com/WordPress/gutenberg/pull/80105) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This PR fixes a readability regression in the DataViews popover filter dropdown where list item text and descriptions turned white on hover and focus, making them invisible against the light hover background. The issue was introduced by a WordPress 7.0 core style rule that forces `color: #fff` on interactive list items. The fix explicitly assigns the `--wpds-color-foreground-interactive-neutral` design tokens to override the core fallback and restore proper WCAG contrast.

## Impact

- **Block & plugin developers using DataViews:** No code changes required. The fix ships in the `@wordpress/dataviews` package and WordPress 7.1, automatically resolving the contrast issue in the Site Editor and any custom DataViews implementations.
- **Site owners & editors:** Filter dropdowns in the Site Editor (e.g., Pages, Templates, Posts lists) will now display readable text on hover and focus states.
- **Hosting & platform teams:** No configuration or migration needed. The change is a CSS token override within the DataViews package.
- **Action required:** None.

## Technical details

The change modifies `packages/dataviews/src/components/dataviews-filters/style.scss` to explicitly set text colors for the `.dataviews-filters__search-widget-listitem` class and its nested description element. It replaces the inherited core hover color with official interactive neutral tokens:

- Resting state: `color: var(--wpds-color-foreground-interactive-neutral);`
- Active/focus state: `color: var(--wpds-color-foreground-interactive-neutral-active);`
- Description resting: `color: var(--wpds-color-foreground-interactive-neutral-weak);`
- Description active/focus: `color: var(--wpds-color-foreground-interactive-neutral-weak-active);`

This CSS specificity override prevents the WordPress 7.0 core rule from applying to the popover dropdown. A snippet of the diff shows the targeted overrides:

```scss
.dataviews-filters__search-widget-listitem {
	cursor: default;
	min-height: var(--wpds-dimension-size-md);
	@include body-medium();
+ color: var(--wpds-color-foreground-interactive-neutral);

	&[data-active-item],
	&:focus {
		background-color: var(--wpds-color-background-interactive-brand-weak-active);
+ color: var(--wpds-color-foreground-interactive-neutral-active);
		// ...
		.dataviews-filters__search-widget-listitem-description {
+ color: var(--wpds-color-foreground-interactive-neutral-weak-active);
		}
	}

	.dataviews-filters__search-widget-listitem-description {
- color: var(--wpds-color-foreground-content-neutral-weak);
+ color: var(--wpds-color-foreground-interactive-neutral-weak);
	}
}
```
No PHP hooks, REST routes, or block.json fields are affected.

## Contribution

Opened by first-time contributor @Pranjal1423 to address issue #80067. The PR was reviewed and approved by @ntsekouras, @ciampo, @t-hamano, and @simison. After initial merge to `main`, the change was backported to the Gutenberg 23.6 release candidate and cherry-picked to the `wp/7.1` branch by @t-hamano to ensure it ships with WordPress 7.1. The discussion focused on coordinating the backport timeline across the RC and beta branches.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
