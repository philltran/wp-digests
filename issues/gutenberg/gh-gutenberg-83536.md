# #83536: ESLint: Add allowUseWithCaution option to use-recommended-components

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Type] Enhancement`, `[Tool] ESLint plugin`, `[Package] UI`
- **Merged:** [`087316d`](https://github.com/WordPress/gutenberg/commit/087316d5c56b2856b86bc16f0a21da0df8f4f353)
- **Discussion:** [#83536](https://github.com/WordPress/gutenberg/pull/83536) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `use-recommended-components` ESLint rule in `@wordpress/eslint-plugin` gains an `allowUseWithCaution` option (default `false`). When enabled, 19 `@wordpress/ui` components marked "Use with caution" in Storybook—such as `Button`, `Dialog`, `Popover`, and `Switch`—are no longer flagged. Previously, any `@wordpress/ui` import not in the stable allowlist was an error, blocking external consumers (plugins, SPAs) from adopting the newer package even when they were willing to track API changes.

## Impact

- **Plugin & theme developers using `@wordpress/eslint-plugin`:** You can now opt in to "Use with caution" components by setting the rule option. No code change is required if you are not importing from `@wordpress/ui` or do not want to use those components.
- **No breaking change.** The option defaults to `false`, so existing lint configurations behave identically.
- **To use the new components:** Add the option to your ESLint config:

  ```js
  '@wordpress/use-recommended-components': ['error', { allowUseWithCaution: true }]
  ```

- **Hosting & platform / headless & REST consumers:** No action required.

## Technical details

In `packages/eslint-plugin/rules/use-recommended-components.js`, the `ALLOWLIST` entry for `@wordpress/ui` gains a `caution` array alongside the existing `allowed` array. The JSDoc type widens from `{ allowed: string[], message?: string }` to `{ allowed: string[], caution?: string[], message?: string }`.

The rule's `schema` changes from `[]` to:

```js
schema: [
  {
    type: 'object',
    properties: {
      allowUseWithCaution: { type: 'boolean' },
    },
    additionalProperties: false,
  },
]
```

In `create()`, the option is destructured: `const { allowUseWithCaution = false } = context.options[0] ?? {};`. The import-specifier check now permits a name if it is in `allowed` **or** if `allowUseWithCaution` is true and the name is in `caution`:

```js
if (
  allowlistEntry &&
  ! allowlistEntry.allowed.includes( name ) &&
  ! (
    allowUseWithCaution &&
    allowlistEntry.caution?.includes( name )
  )
) {
  context.report( { node: specifier, /* … */ } );
}
```

The 19 components in `caution` are: `AlertDialog`, `Breadcrumb`, `Button`, `Checkbox`, `CheckboxGroup`, `Combobox`, `Dialog`, `Drawer`, `IconButton`, `LinkButton`, `Menu`, `Notice`, `Popover`, `Radio`, `RadioGroup`, `SearchableSelect`, `SearchableSelectControl`, `Switch`, `SwitchControl`.

A new unit test asserts that no component name appears in both `allowed` and `caution` for any package entry. The `@wordpress/ui` `CONTRIBUTING.md` gains a "Component status" section instructing contributors to keep the ESLint rule's `allowed`/`caution` lists in sync with Storybook `parameters.componentStatus.status` values.

## Contribution

Opened by @simison with @mirka as co-author. The PR references a companion ticket (#83535) discussing three components (`ControlWithError`, `ValidatedInputControl`, `ValidatedTextareaControl`) that the rule currently allows but Storybook marks as caution—a mismatch deferred to that separate effort. @mirka suggested adding a contributing-doc note about keeping the rule in sync with Storybook status, which @simison incorporated as the new "Component status" section in `packages/ui/CONTRIBUTING.md`. AI tooling was used in authorship (disclosed per project guidelines).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
