# #81471: Editor: Stop `:has()` selectors recalculating the whole document on every block selection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Editor`, `[Package] Edit Post`, `[Package] Edit Site`
- **Merged:** [`7455cde`](https://github.com/WordPress/gutenberg/commit/7455cde4fc17a3228496b4dbc44a72f793dbbec7)
- **Discussion:** [#81471](https://github.com/WordPress/gutenberg/pull/81471) · 13 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Four groups of editor and interface CSS rules were rewritten so that each `:has()` is the subject of its own rule. The old `body:has(.editor-editor-interface.is-distraction-free) #wpadminbar` selector made Blink flag the whole `body` subtree for style recalculation on every DOM change during the React commit. In a 1000-paragraph post, clicking a paragraph triggered about 8 full-document style recalcs. After the change, the reported first-click recalc dropped from 6178 to 476 elements and the median `Selecting blocks` metric from 32.86 ms to 27.32 ms.

## Impact

- **Site owners / editor users:** Block selection in large posts is faster, most noticeably on the first click after load. Distraction-free mode, the revisions sidebar, and the floating notes (collab) sidebar should look and behave as before.
- **Plugin & theme developers:** No API changes. If custom admin CSS or plugin styles target the old descendant selectors (for example `.interface-complementary-area:has(.editor-post-revisions-timeline) .editor-sidebar__panel-tabs`, or `.interface-complementary-area-header` inside the collab sidebar), verify they still match. The rules that hid `#wpadminbar` and the collab header are now expressed differently.
- **Custom CSS authors:** Anyone who overrides `#wpadminbar { display: ... }` under `body.is-fullscreen-mode` or `body.site-editor-php` should note that the rule now reads `display: var(--wp-editor-admin-bar-display, block)`, so specificity and cascade interactions may differ.
- **Hosting / headless / REST:** Not affected.
- **Action required:** None for most sites. Smoke-test distraction-free mode, the revisions sidebar, and the notes sidebar if you ship custom editor styling.

## Technical details

The change is SCSS-only, across `edit-post`, `edit-site`, and `editor`.

**Distraction-free admin bar** (`packages/edit-post/src/style.scss`, `packages/edit-site/src/style.scss`): Previously `body:has(.editor-editor-interface.is-distraction-free)` also contained a nested `#wpadminbar { display: none }`. That made the `:has()` subject a descendant of the anchor, so Blink cannot determine which descendants are affected and invalidates the anchor's entire subtree on each DOM mutation. Now `body:has(...)` only sets custom properties, and a separate `#wpadminbar` rule consumes one of them:

```scss
// before
&:has(.editor-editor-interface.is-distraction-free) {
	--wp-admin--admin-bar--height: 0px;
	#wpadminbar { display: none; }
}

// after
&:has(.editor-editor-interface.is-distraction-free) {
	--wp-admin--admin-bar--height: 0px;
	--wp-editor-admin-bar-display: none;
}
#wpadminbar {
	display: var(--wp-editor-admin-bar-display, block);
}
```

The new custom property is `--wp-editor-admin-bar-display`. In `edit-post` the rules remain inside `&.is-fullscreen-mode` and `break-medium`.

**Collab sidebar** (`packages/editor/src/components/collab-sidebar/style.scss`): The nested `.interface-complementary-area-header { display: none }` under `.interface-skeleton__sidebar:has(.editor-collab-sidebar)` is replaced by a standalone `.editor-collab-sidebar__header { display: none }`. The `:has()` rule now only removes `box-shadow`. The diff does not show the component change that adds the `editor-collab-sidebar__header` class to the header element, so this may be in the PR outside the shown diff or already present.

**Revisions timeline** (`packages/editor/src/components/post-revisions-timeline/style.scss`): The nested rules under `.interface-complementary-area:has(.editor-post-revisions-timeline)` and `.editor-sidebar__panel:has(...)` are unnested into standalone selectors:
- `.interface-complementary-area > .editor-sidebar__panel-tabs`
- `.editor-sidebar__panel [role="tabpanel"]:has(.editor-post-revisions-timeline)`
- `.editor-sidebar__panel .editor-revision-meta-diff__content`
- `.editor-sidebar__panel .editor-post-revisions-timeline`

Per the code comments, the subjects exist only in revisions mode, so the extra `:has()` scoping is not required (the tabs rule is described as inert unless the parent rule applies).

The compressed-size bot reports +80 B total: the edit-post and edit-site CSS grow by roughly 13-17 B each, and the editor CSS shrinks by 7-13 B.

## Contribution

@Mamaduka authored the PR (noting it was assisted by Claude) as a follow-up to #81457. Review discussion turned to the `Selecting blocks` performance spec, which discards the first click and so understates first-selection regressions such as the earlier editable-root change. @youknowriad said he added the throwaway step intentionally long ago and agreed with measuring first selection separately by clearing selection between iterations. That change was proposed for the metrics spec rather than made in this PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
