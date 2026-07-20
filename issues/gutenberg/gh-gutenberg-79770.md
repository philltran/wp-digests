# #79770: Theme: Fill semantic token state gaps

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Edit Site`, `[Package] Base styles`, `[Package] Theme`
- **Merged:** [`d739943`](https://github.com/WordPress/gutenberg/commit/d739943c1593327e4b6ed0e0d240bd8e8baadc0e)
- **Discussion:** [#79770](https://github.com/WordPress/gutenberg/pull/79770) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The WordPress design system (`@wordpress/theme`) now exposes three new semantic CSS custom properties and renames one existing token to better align with its role-based naming convention. Specifically, it adds `--wpds-color-background-thumb-brand-disabled` and `--wpds-color-foreground-interactive-neutral-weak-active`, while renaming `--wpds-color-background-thumb-neutral-disabled` to `--wpds-color-background-thumb-neutral-weak-disabled`. This ensures disabled and active states for neutral/weak emphasis components have explicit, semantically correct tokens instead of relying on generic or mismatched fallbacks.

## Impact

- **Theme & plugin developers:** If you directly reference `--wpds-color-background-thumb-neutral-disabled` in custom CSS or SCSS, you must update it to `--wpds-color-background-thumb-neutral-weak-disabled` to maintain expected styling.
- **Edit Site / Block theme authors:** The internal edit-site sidebar navigation styles now use `--wpds-color-foreground-interactive-neutral-weak-active` for hover/focus/active states, which may slightly adjust contrast in the site editor if you have custom overrides targeting those states.
- **No action required** for most users, as the visual values remain identical and the change primarily improves semantic clarity and parity across the token matrix.

## Technical details

The diff modifies the token generation pipeline across `packages/theme/tokens/color.json`, `packages/theme/src/prebuilt/css/design-tokens.css`, `packages/theme/src/prebuilt/js/design-tokens.mjs`, and `packages/theme/src/prebuilt/ts/token-types.ts`. The renamed token (`--wpds-color-background-thumb-neutral-disabled` → `--wpds-color-background-thumb-neutral-weak-disabled`) and new tokens (`--wpds-color-background-thumb-brand-disabled`, `--wpds-color-foreground-interactive-neutral-weak-active`) are added to the SCSS fallback map in `packages/base-styles/internal/_wpds-token-fallbacks.scss` and exported via the prebuilt JS/TS modules. Internally, `packages/edit-site/src/components/sidebar-navigation-item/style.scss` and `packages/edit-site/src/components/sidebar-navigation-screen-navigation-menus/style.scss` replace `var(--wpds-color-foreground-interactive-neutral-active)` with `var(--wpds-color-foreground-interactive-neutral-weak-active)` for `&:hover`, `&:focus`, and `&[aria-current]` states. The `packages/theme/docs/tokens.md` documentation is updated to explicitly state that the token set is role-based rather than a full combinatorial matrix.

## Contribution

Opened and merged by @ciampo (with co-authorship from @mirka), this PR serves as a follow-up to #77462 to address gaps identified during token documentation. The changes were generated with AI assistance, reviewed by the author, and merged without significant design debate, as the updates strictly align with the existing role-based token architecture and maintain visual parity.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
