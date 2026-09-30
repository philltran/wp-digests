# #83038: UI: Add RadioGroup form primitive

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`08f6033`](https://github.com/WordPress/gutenberg/commit/08f60330ef390b52bb2977ebd34afc1b179db929)
- **Discussion:** [#83038](https://github.com/WordPress/gutenberg/pull/83038) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a `RadioGroup` form primitive to `@wordpress/ui`, wrapping Base UI's `RadioGroup` with a default column `Stack` render. This gives consumers a package-level export for grouping `Radio` items so they share a single selected value, eliminating the need to import Base UI's `RadioGroup` directly. It is part of the broader `@wordpress/ui` form-primitive buildout (tracking issue #74178) and mirrors the pattern established by the recently added `CheckboxGroup`.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:** A new named export `RadioGroup` is available from `@wordpress/ui` (re-exported from `packages/ui/src/form/primitives/index.ts`). It is marked **use-with-caution** in Storybook and is not yet recommended for use alongside `@wordpress/components` components. The higher-level `RadioGroupControl` is described as "coming soon" and will be the primary consumer-facing component.
- **Existing `Radio` primitive users:** No breaking change. The `Radio` component's public API is unchanged; only its JSDoc and the internal story/test imports were updated to reference the new package-level `RadioGroup` instead of `@base-ui/react/radio-group`.
- **No action required** for sites, themes, or plugins that do not import from `@wordpress/ui` form primitives.

## Technical details

The new component lives at `packages/ui/src/form/primitives/radio-group/radio-group.tsx`. It is a `forwardRef<HTMLDivElement, RadioGroupProps>` wrapper around `@base-ui/react/radio-group`'s `RadioGroup`:

```tsx
const DEFAULT_RENDER = ( props: React.ComponentProps< typeof Stack > ) => (
  <Stack { ...props } direction="column" gap="sm" />
);

export const RadioGroup = forwardRef< HTMLDivElement, RadioGroupProps >( function RadioGroup( { render = DEFAULT_RENDER, ...restProps }, ref ) {
  return <_RadioGroup ref={ ref } render={ render } { ...restProps } />;
} );
```

The props type (`types.ts`) is `ComponentProps<typeof _RadioGroup> & { children?: React.ReactNode }`, so all Base UI `RadioGroup` props (`defaultValue`, `value`, `onValueChange`, `disabled`, `name`, `orientation`, etc.) pass through unchanged.

The barrel export in `packages/ui/src/form/primitives/index.ts` gains:

```ts
export { RadioGroup } from './radio-group';
```

The existing `Radio` stories and jsdom test previously imported `RadioGroup` directly from `@base-ui/react/radio-group`; both now import from the new `../../radio-group` path. The `Radio` JSDoc was updated from "Must be rendered inside a RadioGroup." to "Must be rendered inside a `RadioGroup`." (backtick formatting only).

## Contribution

Opened by @mirka as part of the #74178 form-primitive tracking issue, following the same pattern as the `CheckboxGroup` primitive added in #82556. @aduth is credited as co-author. The PR carried only 2 comments and no visible design debate; it was a straightforward wrapper addition with stories, a ref-forwarding test, and the import migration in existing `Radio` stories/tests.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
