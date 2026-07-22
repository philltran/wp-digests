# #77646: Tabs: Simplify layout and prune redundant block supports

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Tabs`
- **Merged:** [`09a4cb9`](https://github.com/WordPress/gutenberg/commit/09a4cb9e050a30a17eb34f8989f84bff8cf9d82d)
- **Discussion:** [#77646](https://github.com/WordPress/gutenberg/pull/77646) · 6 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Tabs block family (`core/tabs`, `core/tab-list`, `core/tab-panel`, `core/tab-panels`) has been simplified by stripping redundant block supports and consolidating default layout configurations. The change removes explicit `false` support declarations that duplicated defaults, prunes unused layout toggles, and eliminates the standalone `tab-panels` stylesheet. This reduction aims to stabilize the block's API and reduce CSS payload ahead of future feature expansion.

## Impact

- **Plugin & theme developers:** No immediate code changes required for standard implementations. However, custom CSS or block configurations explicitly targeting the removed layout supports (e.g., `layout.allowOrientation`, `layout.allowVerticalAlignment`, `layout.allowJustification`, `layout.allowSizingOnChildren`, `layout.allowWrap`, `layout.allowEditing`) or the deleted `tab-panels` stylesheet will silently stop affecting output.
- **Block authors:** The `block.json` `supports` definitions for the tab blocks now omit several previously documented `false` values and layout toggles. Custom blocks inheriting or extending these supports should verify their configurations against the new defaults.
- **No action required** for end users or standard theme/plugin deployments.

## Technical details

The diff modifies `packages/block-library/src/*/block.json` and associated stylesheets to prune explicit `false` support declarations and simplify `layout.default` objects. Key changes include:
- `tab-list/block.json`: `layout.default` changed from `{"type":"flex","flexWrap":"nowrap","orientation":"horizontal"}` to `{"type":"flex"}`. Removed `allowSwitching`, `allowVerticalAlignment`, `allowJustification`, `allowOrientation`, `dimensions`, `spacing.margin`, and `reusable`.
- `tab-panels/block.json`: Removed the entire `layout` support object, along with `anchor`, `dimensions`, `spacing.blockGap`, `spacing.margin`, and `reusable`. The `editorScript` and `style` file references were replaced with registered handles (`wp-block-tab-list-editor`, `wp-block-tab-list`, `wp-block-tab-panel`).
- `tab-panel/block.json`: Removed `renaming`, `spacing.margin`, and `reusable`. Updated `style` reference to `wp-block-tab-panel`.
- `tabs/block.json`: Replaced complex flex layout defaults with `layout.allowEditing: false`. Removed `renaming`.
- CSS cleanup: `packages/block-library/src/tab-panels/style.scss` was deleted, and its import removed from `style.scss`. `tab-panel/style.scss` lost flexbox sizing rules and background padding overrides.
- Documentation in `README.md` files was updated to match the new `supports` schema.

Before/after layout default example:
```json
// Before (tab-list)
"layout": {
  "default": {"type":"flex","flexWrap":"nowrap","orientation":"horizontal"},
  "allowSwitching": false,
  "allowVerticalAlignment": true,
  "allowJustification": true,
  "allowOrientation": true
}

// After (tab-list)
"layout": {
  "default": {"type":"flex"},
  "allowVerticalAlignment": false,
  "allowOrientation": false,
  "allowWrap": false
}
```

## Contribution

Authored by @t-hamano with review and co-authorship from @carolinan, @fabiankaegy, and @jasmussen. The PR was driven by a stabilization goal: the author noted that removing features post-stabilization is harder than adding them later, so the initial release was narrowed to essential layout options. Reviewers supported the simplification to reduce complexity and CSS bloat, with no major alternative approaches debated before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
