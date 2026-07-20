# #80024: Packages: Widen React peer dependency support to include React 19

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] Enhancement`, `[Package] Data`, `[Package] Server Side Render`, `[Package] Compose`, `[Package] Core data`, `[Package] Viewport`, `[Package] Keycodes`, `[Package] Plugins`, `[Package] Components`, `[Package] Blocks`, `[Package] Editor`, `[Package] Block library`, `[Package] Notices`, `[Package] Format library`, `[Package] Rich text`, `[Package] Block editor`, `[Package] Edit Post`, `[Package] Data Controls`, `[Package] Edit Widgets`, `[Package] Interface`, `[Package] Primitives`, `[Package] Icons`, `[Package] Keyboard Shortcuts`, `[Package] Edit Site`, `[Package] React i18n`, `[Package] Style Engine`, `[Package] Preferences`, `[Package] Media Utils`, `[Package] Commands`, `[Package] Core commands`, `[Package] Patterns`, `[Package] DataViews`, `[Package] Fields`, `[Package] Router`, `[Package] Theme`, `[Package] UI`
- **Merged:** [`28e3787`](https://github.com/WordPress/gutenberg/commit/28e3787b8d15332b2eff0f6235574394a8ee4393)
- **Discussion:** [#80024](https://github.com/WordPress/gutenberg/pull/80024) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg monorepo has updated the `peerDependencies` across its published `@wordpress/*` packages to explicitly support React 19 alongside React 18. Previously, all packages restricted React to `^18.0.0`, which triggered npm/yarn warnings or installation failures for downstream consumers already using React 19. This change aligns the package manifests with the experimental React 19 support introduced in the editor.

## Impact

- **Plugin & theme developers**: No code changes required. Projects using React 19 will no longer see peer dependency warnings when installing `@wordpress/*` packages.
- **Hosting & platform teams**: No action required. The runtime behavior of the packages remains unchanged.
- **Headless & REST consumers**: No impact; this is strictly a package manifest update.

## Technical details

The diff updates the `peerDependencies` field in `package.json` files across dozens of `@wordpress/*` packages (e.g., `packages/block-editor`, `packages/components`, `packages/editor`, `packages/rich-text`). The version constraint for `react`, `react-dom`, `@types/react`, and `@types/react-dom` is changed from `^18.0.0` (or `^18.3.27` for types) to `^18 || ^19`. 

Before:
```json
"peerDependencies": {
  "react": "^18.0.0",
  "react-dom": "^18.0.0"
}
```

After:
```json
"peerDependencies": {
  "react": "^18 || ^19",
  "react-dom": "^18 || ^19"
}
```

The change is purely declarative in the package manifests and does not modify any JavaScript/TypeScript source files, hooks, or runtime APIs.

## Contribution

Opened and merged by @aduth, with co-authors @manzoorwanijk, @jsnajdr, @tyxla, and @sirlouen. The PR was a straightforward manifest update to align with the experimental React 19 flag introduced in #79077. The author used Cursor + Composer to automate the `package.json` and `CHANGELOG.md` updates, followed by manual verification. No alternative approaches or significant design debates were recorded in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
