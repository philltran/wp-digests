# #79657: Radio: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`
- **Merged:** [`b4b7b46`](https://github.com/WordPress/gutenberg/commit/b4b7b469de595c96e994000c85c12978fb6abeb8)
- **Discussion:** [#79657](https://github.com/WordPress/gutenberg/pull/79657) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `Radio` component in `@wordpress/components` hard-deprecates the `__next40pxDefaultSize` prop, ending the opt-in period for its larger default sizing. The prop is now stripped at the component boundary and hardcoded onto the inner `<Button />`, making the larger 40px height the permanent default behavior. Developers should remove the prop from all call sites to maintain clean code and prepare for potential future removal of the argument entirely.

## Impact

- **Block/plugin/theme developers**: Remove `__next40pxDefaultSize` from `<Radio />` and `<RadioGroup />` usage. Visual output remains unchanged, but keeping the prop will trigger deprecation notices in tooling.
- **Editor maintainers & internal teams**: Update any internal implementations or override styles that previously relied on conditionally applying this prop.
- **No action required** for end-users or external API consumers, as the sizing change is entirely internal to `@wordpress/components`.

## Technical details

The diff modifies `packages/components/src/radio-group/radio.tsx` to destructure and discard `__next40pxDefaultSize` from props, then applies it directly to the inner `<Button />` component. The previous `maybeWarnDeprecated36pxSize` warning call is removed. TypeScript definitions in `packages/components/src/radio-group/types.ts` now explicitly mark `RadioProps.__next40pxDefaultSize` as deprecated (default since WP 7.1) and tag it with `@ignore`. Corresponding ESLint configuration updates remove `'Radio'` from the `COMPONENTS_REQUIRING_40PX` set in `packages/eslint-plugin/rules/components-no-missing-40px-size-prop.js` and add it to a restricted syntax list in `tools/eslint/config.mjs`.

Before/after usage pattern:
```diff
// Before
<Radio __next40pxDefaultSize value="25">25%</Radio>
<Radio __next40pxDefaultSize value="50">50%</Radio>

// After
<Radio value="25">25%</Radio>
<Radio value="50">50%</Radio>
```

## Contribution

Opened and merged by @mirka (co-authored by @ciampo) as part of an ongoing Gutenberg component sizing cleanup. The PR was streamlined toward finalizing an opt-in period for a larger default button height, with minimal review friction. Discussion centered on standard size-change verification and bundle metrics, and the change was merged after passing automated checks.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
