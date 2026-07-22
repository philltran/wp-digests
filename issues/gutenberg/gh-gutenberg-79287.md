# #79287: Site Editor: Save hub button styling while saving

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Bug`, `Needs Design Feedback`, `[Package] Edit Site`, `[Feature] Site Editor`
- **Merged:** [`30ad25c`](https://github.com/WordPress/gutenberg/commit/30ad25c73385e37247f16b05fc9bf2c557b57c8f)
- **Discussion:** [#79287](https://github.com/WordPress/gutenberg/pull/79287) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Site Editor’s save hub button now correctly retains its `primary` variant styling while a save operation is in progress. Previously, the button would lose its variant class during the save state, resulting in a grey, low-contrast appearance. This fix ensures the button maintains proper visual hierarchy and readability throughout the save lifecycle.

## Impact

- **Site editors & content creators:** No code changes required. The UI behavior is corrected automatically upon updating to this version.
- **Plugin & theme developers:** No impact. The change is isolated to the Site Editor’s internal `SaveHub` component and does not expose new APIs or modify existing block/theme interfaces.
- **No action required.**

## Technical details

The diff modifies `packages/edit-site/src/components/save-hub/index.js` to fix the button’s variant logic and migrate layout components. The `variant` prop on `SaveButton` was updated from:
```js
variant={ isDisabled ? null : 'primary' }
```
to:
```js
variant={ isDisabled && ! isSaving ? null : 'primary' }
```
This ensures the `primary` variant persists when `isSaving` is true. The component also replaces `__experimentalHStack` from `@wordpress/components` with `Stack` from `@wordpress/ui`, updating layout props from `alignment="right" spacing={ 4 }` to `gap="lg"`. Corresponding SCSS padding in `style.scss` was adjusted, and the file’s ESLint suppression for `@wordpress/use-recommended-components` was removed from `tools/eslint/suppressions.json`.

## Contribution

Opened to address a visual regression where the save button appeared broken during save operations. During review, a contributor suggested migrating the deprecated `__experimentalHStack` to the newer `Stack` component from `@wordpress/ui` to resolve a suppressed ESLint warning. The author implemented the migration, and the PR was merged after confirming the original behavior was unintentional rather than a deliberate design choice.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
