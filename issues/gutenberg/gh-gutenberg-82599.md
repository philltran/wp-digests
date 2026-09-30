# #82599: Editor: Derive post-only block list layout from the assigned template

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Bug`, `[Package] Editor`, `[Feature] Site Editor`
- **Merged:** [`0c01e1d`](https://github.com/WordPress/gutenberg/commit/0c01e1d3d6985e905fc474d04b3f7d7263e66c6a)
- **Discussion:** [#82599](https://github.com/WordPress/gutenberg/pull/82599) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

In the site editor's post-only mode ("Show template" off), the root block list layout is now derived from the Content block of the template actually assigned to the post. Previously it fell back to the theme.json global layout whenever the server-provided `postContentAttributes` editor setting was absent, so the alignment dropdown offered Wide and Full width even when the template's Content block didn't support them. Those choices appeared to work in the canvas but had no effect on the front end.

## Impact

**Site owners / editors**
- When editing a page in the site editor with the template hidden, the alignment options now match what the template supports. Wide and Full are no longer offered when the template's Content block has "Inner blocks use content width" off, and the options are the same as with "Show template" on.

**Theme and plugin developers**
- No API changes and no action required. Only the layout given to the root block list changes, not how alignments are resolved.
- Templates that don't use content width will now show a narrower alignment set in post-only mode. This is the intended behavior.
- Known limitation: a Content block nested inside a Template Part or a `core/pattern` reference is not detected, because neither core's `parse_blocks()` nor the editor's `parse()` expands those. Such templates still resolve to no Content block.

**Headless / REST consumers**
- Not affected.

## Technical details

The change is confined to `packages/editor/src/components/visual-editor/index.jsx`, in two parts.

1. **`blockListLayout` gate.** The ternary now reads `newestPostContentAttributes` instead of the `postContentAttributes` editor setting. The code comment says that setting is built server-side from `global $post_ID`, which is not the post being edited in the site editor, so it never arrives there. `postContentLayout` was already derived from `newestPostContentAttributes`, which resolves the template through `getCurrentTemplateId()`, so the ternary was checking a different variable from the one its own branch is built on.

2. **`newestPostContentAttributes` return value.** It no longer ends in `|| {}`, so it returns `undefined` when the template has no Post Content block, rather than a truthy empty object. Without this, classic themes or templates without a Content block would take the constrained path instead of `fallbackLayout`. Per the PR description, the three consumers (`useLayoutClasses`, `useLayoutStyles`, and the destructure below) already default to `{}`, so `undefined` is safe.

```diff
-return getPostContentAttributes( parse( parseableContent ) ) || {};
+return getPostContentAttributes( parse( parseableContent ) );
...
-const blockListLayout = postContentAttributes
+const blockListLayout = newestPostContentAttributes
 	? postContentLayout
 	: fallbackLayout;
```

## Contribution

@jasmussen authored the PR (written with Claude Opus 5, per the PR's AI disclosure), following up on a comment in issue #79285. @andrewserong reviewed it and found a separate bug, which he confirmed was unrelated and fixed in #82637. Joen merged it as a small change and said he'd watch for follow-ups.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
