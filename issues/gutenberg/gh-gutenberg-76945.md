# #76945: DataViews: Fix wrapper height resolution in flex layouts

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @allilevine
- **Labels:** `[Type] Bug`, `First-time Contributor`, `[Package] DataViews`
- **Merged:** [`21069e8`](https://github.com/WordPress/gutenberg/commit/21069e816c3670039e2c5e64ba3750e29d1f63ae)
- **Discussion:** [#76945](https://github.com/WordPress/gutenberg/pull/76945) · 6 comments · 0 reactions

## Summary

Two CSS declarations — `flex-grow: 1` and `min-height: 0` — were added to the `.dataviews-wrapper` and `.dataviews-picker-wrapper` selectors in the `@wordpress/dataviews` package. The bug: `height: 100%` on `.dataviews-wrapper` cannot resolve to a definite value when the parent's height is established by `flex-grow` rather than an explicit `height` property. Without a definite height on the wrapper, the inner `.dataviews-layout__container`'s `overflow: auto` never activates, so the table expands to its full content height and overflows the viewport instead of scrolling. The fix establishes a definite height in flex contexts without altering behaviour in non-flex layouts.

## Impact

**Plugin & theme developers / `@wordpress/dataviews` consumers**
- If you embed `<DataViews>` inside a flex-column container whose height comes from `flex-grow` (not an explicit `height`), the internal table scroll now works correctly. No code changes required — the fix is in the package CSS.
- The immediate upstream trigger was the `@wordpress/admin-ui` `Page` component, where `.admin-ui-page__content` uses `flex-grow: 1`.

**All other consumers**
- No action required. DataViews in non-flex contexts (e.g., the Site Editor pages list) is unaffected — `height: 100%` is preserved alongside the new declarations.

## Technical details

The fix is CSS-only. Two properties were appended to the existing rule blocks for `.dataviews-wrapper` and `.dataviews-picker-wrapper` in the `@wordpress/dataviews` package source styles:

```css
/* Before */
.dataviews-wrapper {
    height: 100%;
    /* no definite height when parent uses flex-grow → overflow: auto on
       .dataviews-layout__container never activates */
}

/* After */
.dataviews-wrapper {
    height: 100%;   /* retained for non-flex contexts */
    flex-grow: 1;   /* fills remaining space in a flex parent, providing a
                       definite height that height: 100% can resolve against */
    min-height: 0;  /* overrides the browser default min-height: auto on flex
                       items so the element can shrink below content height,
                       allowing overflow: auto to engage */
}
```

The same two declarations were applied to `.dataviews-picker-wrapper`. The compiled output surfaces in `build/styles/edit-site/style*.css` and `build/styles/editor/style*.css` (+51 B across all variants including RTL and minified). No JavaScript, hooks, REST schema, or block APIs were touched.

## Contribution

Opened and merged as PR #76945 (closing issue #76944) by first-time contributor @allilevine, merged at commit `21069e8`. The PR description notes the change was authored with Claude Code assistance and validated by a human. Reviewers credited at merge include @ntsekouras, @jameskoster, and @jasmussen. The discussion thread contains no recorded debate over alternative approaches.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
