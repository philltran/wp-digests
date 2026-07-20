# #79967: BorderBoxControl: Fix unlink button positioning after View Emotion migration

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Bug`, `[Feature] UI Components`, `[Package] Components`
- **Merged:** [`67fda3a`](https://github.com/WordPress/gutenberg/commit/67fda3af310b6945e89297d6615b3391bfc79480)
- **Discussion:** [#79967](https://github.com/WordPress/gutenberg/pull/79967) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a visual regression in the `BorderBoxControl` component where the link/unlink button overlapped the border slider instead of sitting in the right-hand gutter. The regression was introduced when the `View` component was migrated away from Emotion, which changed how `className` styles are injected and caused `BorderControl`'s base `margin: 0` to override the gutter's `margin-right: 24px`. The fix increases selector specificity to ensure the gutter margin is preserved regardless of stylesheet insertion order.

## Impact

- **Block & UI developers:** Affects anyone using or extending `BorderBoxControl` in the block editor sidebar. The link/unlink button will now correctly position itself in the gutter.
- **Plugin & theme developers:** No direct impact unless you are building custom UI components that rely on the `@wordpress/components` `BorderBoxControl` or are affected by the underlying `View`/`useContextSystem` styling behavior.
- **Site owners & end users:** No action required. This is an internal block editor UI fix with no effect on frontend rendering or site configuration.

## Technical details

The change modifies `packages/components/src/border-box-control/styles.ts`. After the `View` migration (#79443), `useContextSystem` injects passed `className` styles before the component's base styles. Because `linkedBorderControl` and `borderControl` land on the same `fieldset` with equal specificity, the cascade winner is now determined by insertion order, causing `borderControl`'s `margin: 0` to win. The fix wraps the styles in `&& { ... }`, doubling the Emotion selector specificity so `marginRight: '24px'` deterministically overrides the base margin.

Before:
```ts
export const linkedBorderControl = () => css`
	flex: 1;
	${ rtl( { marginRight: '24px' } )() }
`;
```

After:
```ts
export const linkedBorderControl = () => css`
	&& {
		flex: 1;
		${ rtl( { marginRight: '24px' } )() }
	}
`;
```

## Contribution

Opened and merged by @aaronrobertshaw, with co-authorship from @ramonjd. The fix addresses a regression introduced in PR #79443. During review, @ciampo noted that while the `&&` specificity bump resolves the immediate issue, a longer-term solution involves migrating affected components to CSS Modules to eliminate Emotion's insertion-order dependency. Follow-up work (#80437, #80445) was already in progress to address similar regressions and begin the CSS Modules migration.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
