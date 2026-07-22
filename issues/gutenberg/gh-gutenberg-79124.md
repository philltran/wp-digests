# #79124: Theme: Add disabled variants for brand and error interactive color tokens

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Theme`, `[Package] UI`
- **Merged:** [`cd7b4af`](https://github.com/WordPress/gutenberg/commit/cd7b4afed6c2c9e5aa179043e940b467ae2e2ff6)
- **Discussion:** [#79124](https://github.com/WordPress/gutenberg/pull/79124) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg theme package now exposes `disabled` CSS custom properties for `brand` and `error` interactive color tokens, bringing them to parity with the existing `neutral` disabled tokens. Previously, disabled interactive elements in these tones had to fall back to neutral disabled values. This change adds explicit token definitions and updates core UI components to reference them, ensuring consistent design system semantics without altering visual output.

## Impact

- **Plugin & theme developers:** No action required. The new CSS variables are additive and follow the existing naming convention.
- **UI component consumers:** Components that previously relied on neutral tokens for disabled states will now reference their respective tone-specific disabled tokens, though the rendered colors remain identical.
- **Design system & tooling:** Eliminates the need to borrow `neutral` disabled tokens for brand/error interactive elements, improving token documentation accuracy and reducing ambiguity for AI agents or automated token parsers.

## Technical details

The change modifies `packages/theme/tokens/color.json` to define new disabled variants for `background.interactive`, `foreground.interactive`, and `stroke.interactive` across `brand` and `error` tones. These definitions are compiled into `packages/theme/src/prebuilt/css/design-tokens.css`, adding variables such as `--wpds-color-background-interactive-brand-strong-disabled` and `--wpds-color-foreground-interactive-error-disabled`. The PR also updates `@wordpress/ui` components (`Button`, `AlertDialog`, and disabled `Select`/`Autocomplete`/`Combobox` list items) to reference these new tokens when in a disabled state. Documentation in `packages/theme/docs/tokens.md` was regenerated to reflect the expanded token table, and the `CHANGELOG.md` was updated. No PHP hooks, filters, or REST schema changes are involved; this is strictly a CSS/design-token and React component update.

## Contribution

The PR was authored and merged following a brief review exchange regarding token completeness. A reviewer questioned whether duplicate tokens should be added for the `content` tone family, prompting clarification that `content` tokens are intentionally scoped to non-interactive UI and therefore excluded from disabled variants. The team also confirmed that omitting `brand` for non-interactive text aligns with the design system's current scope. The change shipped as a straightforward parity enhancement with no alternative approaches considered or rejected.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
