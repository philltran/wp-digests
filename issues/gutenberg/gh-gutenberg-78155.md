# #78155: migrated __experimentalText, __experimentalHStack, and __experimentalVStack to Text and Stack

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Raxen001
- **Labels:** `[Type] Code Quality`, `[Package] Block editor`, `First-time Contributor`
- **Merged:** [`ac98e90`](https://github.com/WordPress/gutenberg/commit/ac98e90be2a4f30d9563401456cb1c4114a370c7)
- **Discussion:** [#78155](https://github.com/WordPress/gutenberg/pull/78155) · 9 comments · 1 reactions

## Summary

The `BlockCard` component in `@wordpress/block-editor` has been migrated away from three deprecated `@wordpress/components` experimental exports — `__experimentalText`, `__experimentalHStack`, and `__experimentalVStack` — to their stable counterparts `Text` and `Stack` from `@wordpress/ui`. This is part of the broader effort tracked in issue #77265 to normalise text styling and layout primitives across the block and site editors. The visual output is functionally identical; the only intentional behavioural difference is a line-height adjustment inherited from the `@wordpress/ui` `Text` component, which improves consistency with the design system.

## Impact

**Plugin & theme developers**
- No action required for plugin/theme code that *consumes* the block editor UI — this is an internal implementation change inside `packages/block-editor/src/components/block-card/index.js`.
- Projects that *import* `__experimentalText`, `__experimentalHStack`, or `__experimentalVStack` directly from `@wordpress/components` are unaffected by this specific PR, but should be aware that these symbols are on a deprecation path and the recommended replacements are `Text` and `Stack` from `@wordpress/ui`.

**Hosting & platform**
- No action required.

**Headless & REST consumers**
- No action required.

## Technical details

In `packages/block-editor/src/components/block-card/index.js`, the import of `__experimentalText as WCText`, `__experimentalVStack as VStack`, and `__experimentalHStack as HStack` from `@wordpress/components` is replaced with a single named import of `{ Text, Stack }` from `@wordpress/ui`.

The JSX changes replace the layout primitives as follows:

```jsx
// Before
<VStack>
  <HStack justify="flex-start" spacing={ 0 }>
    …
    <VStack spacing={ 1 }>
      …
    </VStack>
  </HStack>
  <WCText className="block-editor-block-card__description">
    { description }
  </WCText>
</VStack>

// After
<Stack direction="column" gap="sm">
  <Stack direction="row" align="center" justify="flex-start">
    …
    <Stack direction="column" gap="xs">
      …
    </Stack>
  </Stack>
  <Text className="block-editor-block-card__description">
    { description }
  </Text>
</Stack>
```

The `spacing` prop used by the `@wordpress/components` experimental stack variants is replaced by the `gap` prop accepted by `@wordpress/ui`'s `Stack`. The outer `VStack` uses `gap="sm"` and the inner `VStack` (wrapping block title and description) uses `gap="xs"`, chosen to visually match the previous spacing after reviewer feedback.

Three suppressions for the `@wordpress/use-recommended-components` ESLint rule were removed from `tools/eslint/suppressions.json` for `packages/block-editor/src/components/block-card/index.js`, since the violations no longer exist.

## Contribution

Opened and authored by first-time contributor @Raxen001 as part of the coordinated experimental-API cleanup tracked in issue #77265. @simison flagged the missing `gap` prop during review, @jameskoster confirmed `gap="sm"` was the correct size to preserve visual parity, and @ciampo drove two rounds of minor feedback before approving. Merged at commit `ac98e90`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
