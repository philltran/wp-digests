# #79590: RangeControl: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Package] DataViews`
- **Merged:** [`5aafb79`](https://github.com/WordPress/gutenberg/commit/5aafb79203a53a41c481ea0de5ba6df26619619b)
- **Discussion:** [#79590](https://github.com/WordPress/gutenberg/pull/79590) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `RangeControl` component permanently adopts the 40px default height, hard-deprecating the legacy `__next40pxDefaultSize` prop. This scheduled cleanup follows a soft deprecation introduced in WordPress 6.7 and fulfills the removal timeline ahead of version 7.1, ensuring consistent visual sizing across editor controls while stripping legacy size-opt-in logic from the public API.

## Impact

- **Plugin & theme developers**: No functional behavior changes; controls render at the same 40px height as before. Developers explicitly passing `__next40pxDefaultSize={ true }` to `<RangeControl>` or `<FlexedRangeControl>` can safely remove the prop.
- **Block & Editor UI developers**: Internal core usages across `block-editor`, `block-library`, and `components` are stripped of the prop. The ESLint rule `components-no-missing-40px-size-prop` is removed for this component, replaced by a `restricted-syntax` rule that blocks passing the prop to prevent runtime warnings.
- **No action required** for end-users or typical plugin integrations relying on default component behavior.

## Technical details

In `packages/components/src/range-control/index.tsx`, `__next40pxDefaultSize` is extracted and ignored to prevent prop drilling into the internal `<input>` element, while all runtime processing of the size flag is removed. The `@deprecated` + `@ignore` TypeScript type stub on `RangeControlProps` is retained for backward compatibility. Legacy warnings from `maybeWarnDeprecated36pxSize` (in `packages/components/src/utils/deprecated-36px-size`) are eliminated. Monorepo usages across `block-editor` (e.g., `height-control/index.js`, `zoom-dropdown.js`) and `block-library` (e.g., `font-size-picker/index.tsx`, `box-input-control.tsx`, `avatar/edit.js`, `query-pagination-numbers/edit.js`) are stripped of the prop entirely. The 40px height is now hardcoded directly on the root element and embedded number input.

## Contribution

Authored by @mirka with co-authorship from @ciampo, this PR serves as the scheduled hard-deprecation step for #65751. Merged as commit `5aafb79`, it represents routine framework maintenance rather than a functional redesign. Review focused on verifying that internal monorepo usages were cleaned up simultaneously and that the TypeScript type stub preserved backward compatibility without triggering runtime warnings.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
