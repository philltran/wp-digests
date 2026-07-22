# #79466: BoxControl: Respect a supplied placeholder via inputProps

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Bug`, `[Feature] UI Components`, `[Package] Components`
- **Merged:** [`843f13d`](https://github.com/WordPress/gutenberg/commit/843f13d00fc195c508c7ef65c7b7c196cd64f96e)
- **Discussion:** [#79466](https://github.com/WordPress/gutenberg/pull/79466) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `BoxControl` component now correctly respects a `placeholder` value passed through its `inputProps` prop. Previously, the component unconditionally overrode any consumer-supplied placeholder with an internal state value, even when all sides shared the same value. This fix ensures the placeholder falls back to the consumer's value when no mixed state exists, enabling UI patterns that display inherited or default values in empty box inputs.

## Impact

- **Block & UI component developers:** No action required. The change fixes a silent prop override but does not alter the public API or break existing implementations.
- **Core/Plugin authors:** No migration needed. The `inputProps` interface already supported `placeholder`; this simply makes the component honor it.
- **Consumers of `BoxControl`:** Can now reliably pass `inputProps={{ placeholder: '...' }}` to show fallback text in empty inputs without the value being stripped.

## Technical details

In `packages/components/src/box-control/input-control.tsx`, the `BoxInputControl` function now destructures `placeholder` from the spread `inputProps` as `placeholderProp`. The internal `placeholder` attribute passed to the underlying input is changed from `placeholder={ mixedPlaceholder }` to `placeholder={ mixedPlaceholder ?? placeholderProp }`. This preserves the "Mixed" placeholder when sides differ, but falls back to the consumer-supplied value when `mixedPlaceholder` is falsy. A unit test was added to `packages/components/src/box-control/test/index.tsx` verifying that `inputProps={{ placeholder: 'Inherited' }}` correctly renders on the textbox role.

## Contribution

Opened to resolve a prop-swallowing bug that blocked work on a related PR (#77894). Reviewed and approved by @ramonjd and @andrewserong, with @andrewserong co-authoring the final implementation. The change was straightforward, received quick approval, and merged immediately after passing e2e tests.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
