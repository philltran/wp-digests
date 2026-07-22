# #79505: ui/IconButton: Restore default tooltip delay

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`bf56887`](https://github.com/WordPress/gutenberg/commit/bf56887eb6b088e295c44f7b3d6154d82ed751e5)
- **Discussion:** [#79505](https://github.com/WordPress/gutenberg/pull/79505) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `IconButton` component in `@wordpress/ui` no longer forces an immediate tooltip appearance on hover. Previously, the component wrapped its internal tooltip in a `Tooltip.Provider` with `delay={ 0 }`, causing tooltips to pop up instantly. This change removes that override, allowing tooltips to respect the default delay or a parent provider’s delay, which reduces visual noise in dense control surfaces.

## Impact

- **Plugin & theme developers**: `IconButton` tooltips will now have a slight delay on hover instead of appearing instantly. No code changes are required unless you explicitly relied on the instant appearance.
- **Grouped controls**: If you render multiple `IconButton`s in a toolbar, menu, or control group, wrap them in a `Tooltip.Provider` to coordinate tooltip behavior across the group.
- **No breaking API changes**: The component signature, props, and export path remain identical; only internal tooltip timing behavior shifts.

## Technical details

The diff modifies `packages/ui/src/icon-button/icon-button.tsx` by stripping the `<Tooltip.Provider delay={ 0 }>` wrapper from the component’s render tree. `IconButton` now renders directly with `<Tooltip.Root>`, `<Tooltip.Trigger>`, and `<Tooltip.Popup>`, inheriting delay behavior from the nearest ancestor provider. A JSDoc block was added to `IconButton` documenting that consumers should wrap grouped instances in `Tooltip.Provider` to manage shared delays. The Storybook story `packages/ui/src/icon-button/stories/index.story.tsx` was updated to demonstrate this grouping pattern.

Before/after render structure:
```tsx
// Before
<Tooltip.Provider delay={ 0 }>
  <Tooltip.Root>
    <Tooltip.Trigger ... />
    <Tooltip.Popup ... />
  </Tooltip.Root>
</Tooltip.Provider>

// After
<Tooltip.Root>
  <Tooltip.Trigger ... />
  <Tooltip.Popup ... />
</Tooltip.Root>
```

## Contribution

Opened by @ciampo to close issue #79461, the change followed a straightforward review cycle with co-authors @mirka, @aduth, @fcoveram, and @shaunandrews. The record shows no significant design debates or alternative approaches; the fix aligns directly with the base UI library’s default behavior and was merged after standard verification steps.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
