# #80099: Theme: Remove elevation tokens

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Base styles`, `[Package] Theme`, `[Package] UI`
- **Merged:** [`716c6e9`](https://github.com/WordPress/gutenberg/commit/716c6e98eba9ed3777323aa555f30d4dad90f243)
- **Discussion:** [#80099](https://github.com/WordPress/gutenberg/pull/80099) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/theme` package has removed the `--wpds-elevation-{xs,sm,md,lg}` CSS custom properties and their underlying token definitions. This change strips the static black `box-shadow` elevation contract from the public design token registry to prevent locking in an incomplete model while the broader elevation system (tracked in #75266) is still being designed. Existing UI components retain their visual shadows by migrating to private, selector-local variables.

## Impact

- **Plugin & theme developers:** Direct consumers of `--wpds-elevation-*` tokens in custom CSS, block styles, or theme.json will see those variables resolve to `undefined`. You must define your own shadow values or wait for the new elevation system.
- **Core UI / Block Editor consumers:** No visual or behavioral changes. Internal components automatically migrated to local variables.
- **Hosting & platform teams:** No action required. The package remains fully compatible; only the token registry changed.
- **Breaking change:** `--wpds-elevation-{xs,sm,md,lg}` removed from `@wordpress/theme`. No direct replacement is provided yet.

## Technical details

The diff removes `packages/theme/tokens/elevation.json` and its reference in `packages/theme/terrazzo.config.ts`. The Terrazzo build no longer generates `--wpds-elevation-*` variables in `packages/theme/src/prebuilt/css/design-tokens.css`, `packages/theme/src/prebuilt/js/design-tokens.mjs`, or `packages/theme/src/prebuilt/js/design-token-fallbacks.mjs`. Internal consumers in `packages/grid/src/dashboard-grid/grid.module.css` and `packages/grid/src/dashboard-lanes/lanes.module.css` replaced `var(--wpds-elevation-md)` with locally scoped `--_wp-grid-elevation-md` variables. The `@wordpress/design-system-mcp` tool description was updated to drop "elevation" from its listed token categories.

Before/after usage pattern:
```css
/* Before */
.drag-preview-frame {
  box-shadow: var(--wpds-elevation-md);
}

/* After */
.drag-preview-frame {
  --_wp-grid-elevation-md: 0 2px 3px rgba(0, 0, 0, 0.05), 0 4px 5px rgba(0, 0, 0, 0.04), 0 12px 12px rgba(0, 0, 0, 0.03), 0 16px 16px rgba(0, 0, 0, 0.02);
  box-shadow: var(--_wp-grid-elevation-md);
}
```

## Contribution

Opened and merged by @ciampo (with @aduth as co-author/reviewer). The change was driven by ongoing design-system discussions (#75266) to avoid committing to a static shadow-only abstraction. @aduth noted that even a broader "shadow" token class might prove inflexible, reinforcing the decision to remove the tokens entirely until a more adaptable elevation model is finalized.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
