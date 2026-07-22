# #78958: Theme: Promote ThemeProvider to stable API

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] New API`, `[Package] Edit Site`, `[Package] Theme`, `[Package] UI`
- **Merged:** [`3fb683e`](https://github.com/WordPress/gutenberg/commit/3fb683e57380482b06f001a0174033e1c7c0ea2f)
- **Discussion:** [#78958](https://github.com/WordPress/gutenberg/pull/78958) · 15 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/theme` package now exports `ThemeProvider` as a stable public API, removing its previous status as a private API gated behind `@wordpress/private-apis`. This change eliminates the need for consumers to use `unlock()` to access the component, resolving bundling conflicts and stabilizing a core UI primitive ahead of the WordPress 7.1 release cycle.

## Impact

- **Plugin & theme developers:** Existing code that relied on the private API gate will break. Consumers must migrate from the `unlock()` pattern to a direct import.
- **Core & package consumers (`@wordpress/boot`, `@wordpress/edit-site`, `@wordpress/ui`):** Internal consumers have been updated to import `ThemeProvider` directly, removing the `@wordpress/private-apis` dependency from these packages.
- **Breaking changes:** The `privateApis` export and the `lock-unlock.ts` gate in `@wordpress/theme` have been removed. Any external code calling `unlock( themePrivateApis ).ThemeProvider` will fail until updated to `import { ThemeProvider } from '@wordpress/theme'`.
- **Note:** The `@wordpress/theme` package README still carries an "experimental" disclaimer until the broader design system tokens stabilize, but `ThemeProvider` itself is now stable.

## Technical details

The diff removes `@wordpress/private-apis` from the dependency trees of `@wordpress/theme`, `@wordpress/ui`, `@wordpress/boot`, and `@wordpress/edit-site`. In `packages/theme/src/index.ts`, the `privateApis` export is deleted and `ThemeProvider` is added to the default exports. The `packages/theme/src/private-apis.ts` and `packages/theme/src/lock-unlock.ts` files are deleted entirely.

Consumers across `@wordpress/ui` (e.g., `packages/ui/src/alert-dialog/popup.tsx`, `packages/ui/src/dialog/popup.tsx`, `packages/ui/src/popover/popup.tsx`) and `@wordpress/boot` (`packages/boot/src/components/root/index.tsx`) replace the unlock pattern with a direct import:

```diff
-import { privateApis as themePrivateApis } from '@wordpress/theme';
-const { ThemeProvider } = unlock( themePrivateApis );
+import { ThemeProvider } from '@wordpress/theme';
```

The `@wordpress/ui` package also removes its own `lock-unlock.ts` gate, as it no longer needs to unlock theme APIs. TypeScript references to `@wordpress/private-apis` are stripped from `packages/theme/tsconfig.src.json`.

## Contribution

Opened and merged by @aduth, with review and co-authorship from @ciampo. The discussion centered on whether to remove the "experimental" banner from the `@wordpress/theme` package README alongside this stabilization. @ciampo recommended keeping the banner until the broader design system tokens reach version 1.0, allowing this PR to focus strictly on `ThemeProvider`. The team also coordinated with PR #79254 to ensure token-breaking changes were finalized before proceeding, though this stabilization landed independently to meet the WordPress 7.1 development timeline.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
