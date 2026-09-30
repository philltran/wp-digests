# #83040: Components: Deprecate Divider

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Tool] ESLint plugin`
- **Merged:** [`cd25c38`](https://github.com/WordPress/gutenberg/commit/cd25c3864b87a6e5609fe08f6dc31ebe012acc9c)
- **Discussion:** [#83040](https://github.com/WordPress/gutenberg/pull/83040) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `__experimentalDivider` component in `@wordpress/components` is formally deprecated with a runtime warning via `@wordpress/deprecated`, scheduled for removal in version 7.4. The barrel export now resolves to a new adapter in `packages/components/src/divider/deprecated.tsx` that fires the deprecation notice before delegating to the existing render. The recommended alternatives are a `Separator` subcomponent (e.g. `Menu.Separator`) when the surrounding component provides one, or custom CSS using design tokens from `@wordpress/theme`.

## Impact

- **Plugin & theme developers using `__experimentalDivider`:** A console warning (`wp.components.__experimentalDivider is deprecated since version 7.2 and will be removed in version 7.4.`) will now fire on every render. Migrate to a `Separator` subcomponent or custom CSS before the 7.4 removal.
- **Developers using `CardDivider`:** No action required. `CardDivider` continues to use the internal `Divider` export from `packages/components/src/divider/index.ts` with the deprecation suppressed, so no warning is logged in Gutenberg Card screens.
- **ESLint users:** The `use-recommended-components` rule message for `__experimentalDivider` is updated to point to `Separator` subcomponents and design tokens rather than the previous generic "planned for deprecation" text.
- **No breaking change in this release.** The component still renders identically; only the warning is added.

## Technical details

The barrel export in `packages/components/src/index.ts` changes from:

```ts
export { Divider as __experimentalDivider } from './divider';
```
to:

```ts
export { default as __experimentalDivider } from './divider/deprecated';
```

The new file `packages/components/src/divider/deprecated.tsx` defines `UnconnectedDeprecatedDivider`, which calls:

```ts
deprecated( 'wp.components.__experimentalDivider', {
	since: '7.2',
	version: '7.4',
} );
```

then delegates to `UnconnectedDivider` (now exported from `component.tsx`) to avoid a double `contextConnect`. The result is wrapped in `contextConnect(UnconnectedDeprecatedDivider, 'Divider')` and exported as default.

The internal `packages/components/src/divider/index.ts` still exports the original connected `Divider` without the deprecation call, so `CardDivider` and other internal consumers remain silent. The Storybook story is relocated to `Components/Deprecated/Divider` and imports from `../deprecated` directly. The `use-recommended-components` ESLint rule denylist entry for `__experimentalDivider` is updated with the new migration guidance.

## Contribution

Opened by @mirka with @aduth as co-author. In discussion, @mirka explained the design of keeping the original `Divider` intact while adding a separate deprecated adapter: two more components (`Elevation` and `Scrollable`) will follow the same pattern, and `Card` itself is slated for deprecation after its usages are migrated. Moving the implementation into the Card folder now would create a second copy of `Divider` to maintain, so the adapter approach was chosen to avoid duplication until the hard removal date arrives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
