# #79860: Packages: Update Ariakit to 0.4.32

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] DataViews`
- **Merged:** [`153f003`](https://github.com/WordPress/gutenberg/commit/153f003c3fd9d1a26f985cff220b9ac0c7895381)
- **Discussion:** [#79860](https://github.com/WordPress/gutenberg/pull/79860) · 2 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Updates the `@ariakit/react` dependency from `0.4.29` to `0.4.32` across `@wordpress/components` and `@wordpress/dataviews`. This resolves a pressed-state handling bug in Command-based widgets (refs #79727) and incorporates upstream patches for focus management, keyboard navigation, dialog/popover edge cases, combobox item matching, and composite list roving tabindex behavior.

## Impact

- **Plugin & theme developers**: No breaking changes or required code modifications. Relying on `@wordpress/components` or `@wordpress/dataviews` will automatically benefit from improved keyboard/pointer interaction stability in ToggleGroupControl, RadioControl, Combobox, Menu, and Popover components upon next deployment.
- **Headless & REST consumers**: Unaffected by direct API shifts; behavioral improvements apply to the underlying JS component libraries bundled with the packages.
- **Hosting & platform teams**: No action required. The update is contained within the `@wordpress` npm workspace and does not alter build tooling, PHP backends, or server-side rendering logic.

## Technical details

The PR modifies only package metadata and lockfile integrity hashes. `packages/components/package.json` and `packages/dataviews/package.json` update the `@ariakit/react` dependency constraint from `^0.4.29` to `^0.4.32`. Corresponding entries in the root `package-lock.json` are updated for `@ariakit/react`, `@ariakit/react-components`, `@ariakit/react-store`, `@ariakit/react-utils`, `@ariakit/store`, and `@ariakit/utils`, along with their SHA-512 integrity strings. No Gutenberg source files, React hooks, or component implementations were altered; the diff contains only version bumps, integrity updates, and changelog entries marked as `Internal`.

## Contribution

Merged as `153f003` by @ciampo (co-authored with @simison and @talldanwp). The PR was opened to address #79727, applying upstream Ariakit patches from `0.4.30`, `0.4.31`, and `0.4.32`. The author provided a comprehensive mapping of upstream releases to Gutenberg component behavior and ran standard dependency audit scripts (`lint:pkg-json`, `lint:lockfile`, `dependency-audit`) prior to merge. No design alternatives were discussed; the scope was strictly limited to the version bump and verification step.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
