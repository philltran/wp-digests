# #78682: Validate additional CSS on mount

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `Global Styles`
- **Merged:** [`834f7d2`](https://github.com/WordPress/gutenberg/commit/834f7d2f487a4840a899c23273eb67b56aa30dd9)
- **Discussion:** [#78682](https://github.com/WordPress/gutenberg/pull/78682) · 4 comments · 0 reactions

## Summary

The Additional CSS panel in the Site Editor (Appearance → Editor → Styles → Additional CSS) now validates saved CSS immediately when the panel mounts, surfacing error notices without requiring any user interaction. Previously, `useState(null)` initialised the error state as empty on every mount, so invalid CSS that had been saved and navigated away from would appear error-free until the user typed in or tabbed out of the field. The fix replaces that plain initialisation with a lazy initialiser that runs the CSS validation synchronously on mount. The same correction applies to per-block Additional CSS entered via a block's advanced inspector panel.

## Impact

- **Plugin & theme developers / site builders**: No API or data-format change. The only visible difference is that pre-existing invalid CSS now triggers the validation error notice as soon as the Additional CSS panel opens, matching the existing behaviour for newly typed input.
- **No action required**: There are no breaking changes, deprecations, or migration steps.

## Technical details

The change is contained in `packages/block-editor/src/components/global-styles/advanced-panel.js` within the `@wordpress/block-editor` package.

Before, the component initialised its error state with a bare `useState(null)`, so the validated error was always `null` on mount regardless of the current CSS value:

```js
// Before
const [ cssError, setCssError ] = useState( null );
```

After, a lazy initialiser is passed to `useState` so the existing CSS value is validated synchronously during the first render:

```js
// After (illustrative — actual lazy initialiser calls the shared validation logic)
const [ cssError, setCssError ] = useState(
    () => validateCSS( currentCSSValue )
);
```

Alongside the lazy-init fix, two smaller refactors were made to extract and share the validation check between the mount path and the interactive (onChange/onBlur) paths, avoiding duplication. The compiled output (`build/scripts/block-editor/index.min.js`) shrank by 7 bytes, consistent with a net de-duplication. No new hooks, filters, REST schema changes, or database changes are introduced.

## Contribution

Opened and authored by @juanfra. @Mamaduka (George Mamadashvili) reviewed, flagging that `npm run lint:js:prune-suppressions` needed to be run to clear lint suppression warnings before merge. @juanfra applied the fix and the PR was merged at commit `834f7d2`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
