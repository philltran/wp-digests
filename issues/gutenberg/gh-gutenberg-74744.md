# #74744: Fix - Accordion: Text in a closed accordion panel cannot be found via the browser search

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @SohamPatel46
- **Labels:** `[Type] Bug`, `[Package] Block library`, `First-time Contributor`, `[Block] Accordion`
- **Merged:** [`2b4421a`](https://github.com/WordPress/gutenberg/commit/2b4421af49f633482aa408235d64c8a25c485bf0)
- **Discussion:** [#74744](https://github.com/WordPress/gutenberg/pull/74744) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Accordion block now makes text inside closed panels indexable by browser search (`Ctrl+F`). Switching from `aria-hidden`/`inert` to `hidden="until-found"` keeps collapsed panels visually hidden while allowing the browser to scan their content; when a match is found, the panel automatically expands to reveal the text.

## Impact

- **Block & Theme Developers**: No migration required. Note that closed panels are now driven by `hidden="until-found"` rather than `aria-hidden` or `display: none`; custom CSS targeting `[aria-hidden]` or `[inert]` on `.wp-block-accordion-panel` will no longer match.
- **Content Editors/Authors**: Browser find will now locate text in closed panels and automatically open the corresponding panel to reveal it.
- **Hosting & Platform Teams**: No action required. The behavior is fully client-side and gracefully degrades to standard `hidden` or visual collapse in browsers that do not support `until-found`.

## Technical details

- `packages/block-library/src/accordion-panel/edit.js` replaces `'aria-hidden': ! hasSelection` with `hidden: ! hasSelection`, preserving edit-mode visibility when a panel is actively selected.
- `packages/block-library/src/accordion/view.js` adds an `isHidden` computed property that returns the string `'until-found'` when an item is closed, and a `handleBeforeMatch` callback that expands the matched panel (respecting `autoclose` context by closing siblings or setting the target to `isOpen = true`).
- `packages/block-library/src/accordion-item/index.php` updates server-side rendering directives to bind `data-wp-bind--hidden="state.isHidden"` and attach `data-wp-on--beforematch="actions.handleBeforeMatch"`, replacing the previous `data-wp-bind--inert` directive.
- `packages/block-library/src/accordion-panel/style.scss` removes `&[inert]` and `&[aria-hidden="true"]` from the margin-reset selector, relying exclusively on `&[hidden]`.

## Contribution

Opened by first-time contributor @SohamPatel46 to address issue #73443. After review feedback highlighting consolidation with PR #74747, the team aligned on the `hidden="until-found"` approach over alternative implementations. Merged by @t-hamano in commit `2b4421a`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
