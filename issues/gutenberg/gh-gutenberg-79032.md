# #79032: Theme: Add Figma scopes to element size tokens

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `Design System`, `[Package] Theme`
- **Merged:** [`2e1dd9b`](https://github.com/WordPress/gutenberg/commit/2e1dd9ba1351c2a3ac07bc1829ee19e125df3215)
- **Discussion:** [#79032](https://github.com/WordPress/gutenberg/pull/79032) · 2 comments · 0 reactions

## Summary

This change adds Figma design variable scopes to the WordPress Design System's dimension tokens in `packages/theme/tokens/dimension.json`. By appending a `$extensions` block with a `WIDTH_HEIGHT` scope, size variables (`5xs` through `lg`) are now correctly surfaced and selectable in Figma's layer dimension pickers. This alignment ensures designers can apply theme sizing directly within Figma without manual overrides or workarounds.

## Impact

- **Theme & Design System maintainers**: No runtime or frontend changes introduced; token JSON structure requires awareness of additive metadata conventions.
- **WordPress plugin & theme developers**: No action required. The underlying CSS custom properties (`--wpds-dimension-size-*`) and their generated values remain completely unchanged.
- **Figma designers & visual asset consumers**: `wpds-dimension.size.*` variables will now appear in Figma's width/height variable pickers. Note that Figma applies the scope to both dimensions simultaneously, as explicitly documented by the author due to platform limitations.

## Technical details

The unified diff modifies `packages/theme/tokens/dimension.json`, targeting eight entries under `wpds-dimension.size` (`5xs`, `4xs`, `3xs`, `2xs`, `xs`, `sm`, `md`, `lg`). For each token, an `$extensions` block is appended alongside the existing `$description`:\n```json\n"$extensions": {\n  "com.figma.scopes": [ "WIDTH_HEIGHT" ]\n}\n```\nThis mirrors the scoping pattern already established for `surface-width` tokens. The change is strictly metadata injection into the Design Tokens JSON schema and does not affect PHP asset generation, CSS output, or block editor rendering logic. A changelog entry was simultaneously added to `packages/theme/CHANGELOG.md` under the `### Internal` header for version 0.14.0.

## Contribution

Opened by @jameskoster as a direct follow-up to #76545, the PR was merged after a brief review cycle that highlighted the need for standardized Figma scope documentation in token source files (@aduth). The change was co-authored by ciampo and introduced without alternative design debates; the merge focused solely on synchronizing existing dimension tokens with Figma's variable picker requirements.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
