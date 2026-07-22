# #79098: theme: rename `bg`/`fg` design token groups to `background`/`foreground`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Edit Site`, `[Package] DataViews`, `[Package] Theme`, `[Package] UI`
- **Merged:** [`a89ef56`](https://github.com/WordPress/gutenberg/commit/a89ef5688bedf07269aa7f06eba116c931f19da6)
- **Discussion:** [#79098](https://github.com/WordPress/gutenberg/pull/79098) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/theme` package renamed its semantic color token groups from abbreviated `bg` and `fg` prefixes to the fully spelled-out `background` and `foreground`. This changes all corresponding CSS custom properties (e.g., `--wpds-color-bg-surface-neutral` becomes `--wpds-color-background-surface-neutral`) across the Design System and internal WordPress packages. The change improves naming consistency with other tokens like `surface` and aligns with the recent `ThemeProvider` prop rename, making the token system more readable and predictable for developers.

## Impact

- **Plugin & theme developers:** If you reference `@wordpress/theme` design tokens or use the `--wpds-color-bg-*` / `--wpds-color-fg-*` custom properties in your stylesheets, you must update them to the new `--wpds-color-background-*` / `--wpds-color-foreground-*` names. This is a breaking change in the Design System token layer.
- **Figma library maintainers:** Tokens exported to Figma will reflect the new naming convention and require re-importing or manual updates.
- **Core WordPress UI / Block Editor users:** No action required; internal packages (`@wordpress/components`, `@wordpress/dataviews`, `@wordpress/edit-site`, etc.) have already been updated to use the new tokens.

## Technical details

The diff systematically replaces `--wpds-color-bg-` with `--wpds-color-background-` and `--wpds-color-fg-` with `--wpds-color-foreground-` across CSS/SCSS files in `packages/admin-ui`, `packages/boot`, `packages/components`, `packages/dataviews`, `packages/edit-site`, and `packages/grid`. The token definitions in `tokens/color.json` were updated, and the TypeScript type generation in `terrazzo.config.ts` was adjusted to match. Prebuilt artifacts (`design-tokens.css`, `design-tokens.mjs`, `color-tokens.ts`, etc.) were regenerated. Primitive ramp tokens (e.g., `primitive.bg`, `bgFill1`) intentionally retain their abbreviated names as they are internal and not exposed as public design tokens.

Before/after example:
```css
/* Before */
.page { background-color: var(--wpds-color-bg-surface-neutral); color: var(--wpds-color-fg-content-neutral); }
/* After */
.page { background-color: var(--wpds-color-background-surface-neutral); color: var(--wpds-color-foreground-content-neutral); }
```

## Contribution

Opened and merged by @ciampo, with review contributions from @aduth and @jameskoster. The change was intentionally split from the broader `ThemeProvider` prop rename in #79007 to keep that PR focused. During the original discussion, the team debated whether to keep the abbreviated `bg`/`fg` prefixes for brevity, but consensus favored spelling them out to match existing tokens like `surface` and improve readability, especially given that some token names are already lengthy. The PR notes it was authored with an AI coding agent, though all changes were manually reviewed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
