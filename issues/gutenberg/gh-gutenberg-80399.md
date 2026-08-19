# #80399: UI: Derive Base UI direction from WordPress i18n

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`4762fb7`](https://github.com/WordPress/gutenberg/commit/4762fb7ab95eefdcd785d1b445175a7a2b5c746e)
- **Discussion:** [#80399](https://github.com/WordPress/gutenberg/pull/80399) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Direction-sensitive components in the `@wordpress/ui` package now read WordPress's text direction automatically instead of inheriting Base UI's hard-coded LTR default. Base UI seeds its internal direction context to `ltr`, so `Autocomplete`, `Combobox`, `Popover`, `Select`, `Tabs`, and `Tooltip` behaved as LTR in RTL locales — wrong arrow-key order in horizontal `Tabs`, wrong positioning/flip logic for anchored content. A new private `DirectionProvider` util reads `isRTL()` from `@wordpress/i18n` and is wrapped around the six `Root` components whose Base UI internals consume direction, so consumers no longer need to mount and forward a second public direction provider of their own.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:** no action required. RTL keyboard navigation and popover placement now follow the site's locale out of the box. If you were wrapping these components in Base UI's `DirectionProvider` from `@base-ui/react/direction-provider` purely to get RTL working, that wrapper is now redundant and can be removed.
- **Anyone deliberately overriding direction from an ancestor:** the provider is now mounted *inside* each `Root`, so it is the nearest provider in the tree and wins over any `DirectionProvider` you render above the component. Direction for these six components is derived from `isRTL()` and is not overridable by an outer Base UI provider.
- **Test authors:** RTL assertions against these components should now mock `@wordpress/i18n`'s `isRTL` rather than rendering an external `DirectionProvider` — that is exactly how the `Tabs` suite was updated.
- **No public API surface was added.** `DirectionProvider` lives at `packages/ui/src/utils/direction-provider.tsx` and is internal to the package; there is no new export for consumers to adopt.
- **Bundle size:** +1.07 kB across built scripts (per the size-change bot), spread over the editor, block-library, block-editor, and boot bundles.

## Technical details

The new file `packages/ui/src/utils/direction-provider.tsx` exports a private wrapper:

```tsx
import { DirectionProvider as BaseUIDirectionProvider } from '@base-ui/react/direction-provider';
import { isRTL } from '@wordpress/i18n';

function DirectionProvider( { children }: { children: ReactNode } ) {
	return (
		<BaseUIDirectionProvider direction={ isRTL() ? 'rtl' : 'ltr' }>
			{ children }
		</BaseUIDirectionProvider>
	);
}
```

It is applied to exactly six `Root` components — `src/form/primitives/autocomplete/root.tsx`, `src/form/primitives/combobox/root.tsx`, `src/form/primitives/select/root.tsx`, `src/popover/root.tsx`, `src/tabs/root.tsx`, and `src/tooltip/root.tsx` — each of which previously rendered the Base UI root bare, e.g. `return <_Popover.Root { ...props } />;` becoming a `<DirectionProvider>`-wrapped render. In `Tabs`, the new provider is the outermost layer, above the existing `TabsValidationProvider`:

```tsx
// Before
<TabsValidationProvider>
	<_Tabs.Root ref={ forwardedRef } { ...otherProps } />
</TabsValidationProvider>

// After
<DirectionProvider>
	<TabsValidationProvider>
		<_Tabs.Root ref={ forwardedRef } { ...otherProps } />
	</TabsValidationProvider>
</DirectionProvider>
```

Because `isRTL()` is read during render rather than passed as a prop, direction is resolved per-render from the i18n runtime; Storybook's existing direction global still works because its i18n filter forces a remount, and the components now pick that value up without extra story wiring.

The `Tabs` RTL test in `packages/ui/src/tabs/test/index.test.tsx` was rewritten accordingly: the `import { DirectionProvider } from '@base-ui/react/direction-provider'` and the JSX wrapper are gone, replaced by a `jest.mock( '@wordpress/i18n', … )` that stubs `isRTL` (defaulting to `false`, reset in `afterEach`) with `mockedIsRTL.mockReturnValue( true )` in the RTL case. The change is recorded in `packages/ui/CHANGELOG.md` under **Bug Fixes**. No props, types, or public exports changed.

## Contribution

Split out of the larger #79560 by @ciampo, who prepared it with AI assistance in Codex. The design point of contention was where the provider should live: rather than exposing a public direction provider that consumers would have to mount and forward, @jsnajdr suggested wrapping each affected component individually. @ciampo asked @mirka and @jsnajdr to confirm the PR interpreted that correctly, and after a long review gap @jsnajdr signed off, noting he was "pleasantly surprised" the per-component approach was realistic given only a handful of components actually consume direction.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
