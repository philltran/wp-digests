# #81824: UI: Align overlay Trigger props with Base UI

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`3faca44`](https://github.com/WordPress/gutenberg/commit/3faca4436be1719a4539474057fd78d538ea57a9)
- **Discussion:** [#81824](https://github.com/WordPress/gutenberg/pull/81824) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The public TypeScript prop types for `AlertDialog.Trigger`, `Dialog.Trigger`, `Drawer.Trigger`, `Popover.Trigger`, and `Tooltip.Trigger` in `@wordpress/ui` are now derived from the corresponding Base UI Trigger components instead of generic `button` props. The wrappers already forwarded every prop to Base UI at runtime, so behavior is unchanged. The types now expose supported props that were previously hidden, such as per-trigger Tooltip `delay`/`closeDelay` and `closeOnClick`.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:**
  - Trigger components now type-check against Base UI's trigger props, so props like `delay`, `closeDelay`, `closeOnClick`, and `disabled` on `Tooltip.Trigger` are visible and typed.
  - `handle` and `payload` are explicitly omitted from all five Trigger types, since detached triggers are not supported by `@wordpress/ui`. Passing them is now a type error.
  - Because types now come from Base UI, code that relied on the older generic `ComponentProps<'button'>` shape could see type differences; run `tsc` after upgrading.
- **Site owners / hosting / REST consumers:** No impact. Runtime overlay behavior is unchanged.

## Technical details

In each of `packages/ui/src/{alert-dialog,dialog,drawer,popover,tooltip}/types.ts`, `TriggerProps` changes from an `interface` extending `ComponentProps<'button'>` to a type alias:

```ts
// Before
export interface TriggerProps extends ComponentProps< 'button' > {
	children?: ReactNode;
}

// After
export type TriggerProps = Omit<
	ComponentProps< typeof _Dialog.Trigger >,
	'handle' | 'payload'
> & {
	children?: ReactNode;
};
```

`Popover.Trigger` previously also picked `openOnHover | delay | closeDelay` from `_Popover.Trigger.Props`; that `Pick` is now subsumed by the derived type. The `children` documentation override is retained. The PR description says the package's normalized `render`, `className`, and `style` types are preserved; the diff itself only shows the `children` override, so that preservation presumably comes from the package's `ComponentProps` helper. `storybook/components-manifest.yml` gains `closeDelay`, `closeOnClick`, `delay`, and `disabled` under `Tooltip.Trigger`, and `packages/ui/CHANGELOG.md` adds an Enhancements entry. No runtime code changes.

## Contribution

Follow-up to #79560, authored by @ciampo and merged with review input credited to @mirka and @Mamaduka. The record shows no notable design debate; the only discussion is bot output.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
