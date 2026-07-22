# #78427: Navigation: Fix missing 'Add block' option in Link UI when inside template parts

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @DarkMatter-999
- **Labels:** `[Type] Enhancement`, `[Package] Block library`
- **Merged:** [`28a3c3e`](https://github.com/WordPress/gutenberg/commit/28a3c3e0d966f437c9da52b389a821755b13d24a)
- **Discussion:** [#78427](https://github.com/WordPress/gutenberg/pull/78427) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Navigation block’s link editor popover now correctly displays the “Add block” and “Create page” options when the block is nested inside a template part and edited in the Site Editor’s content-only mode. Previously, these actions were hidden because the UI only checked for a `default` editing mode. This restores expected inline editing capabilities without forcing users to exit the template context to edit the template part in isolation.

## Impact

- Site editors & theme developers: No code changes required. The Site Editor UI now behaves consistently whether editing a Navigation block directly or through a template part.
- Plugin & theme developers: No breaking changes. The `canAddBlock` and `canAddPage` props passed to the `LinkUI` component now accept `contentOnly` modes alongside `default`.
- No migration, configuration, or code changes required. Existing implementations continue to work unchanged.

## Technical details

The change modifies `packages/block-library/src/navigation-link/link-ui/index.js`. The `UnforwardedLinkUI` component previously restricted `canAddBlock` and `canAddPage` to `blockEditingMode === 'default'`. The diff updates both conditions to `blockEditingMode !== 'disabled'`, allowing the quick inserter and page creation flow to activate in `contentOnly` mode while remaining disabled in `disabled` mode. The block editor’s native `canInsertBlockType` selector continues to filter the inserter to content blocks only.

Before:
```js
canAddBlock={ blockEditingMode === 'default' }
```

After:
```js
canAddBlock={ blockEditingMode !== 'disabled' }
```

## Contribution

The fix addresses a regression introduced in #71489 and was merged after a brief review cycle. During review, a contributor noted that the `!== 'disabled'` check might be redundant since disabled UI is inaccessible, but the condition was retained for explicit state handling. The change was co-authored and approved without further debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
