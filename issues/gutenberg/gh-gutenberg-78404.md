# #78404: Media: Send Document-Isolation-Policy header on the site preview frame

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `Browser Issues`, `[Feature] Client Side Media`
- **Merged:** [`4780ecd`](https://github.com/WordPress/gutenberg/commit/4780ecd55e3ade565a854c356161f8031efc17c2)
- **Discussion:** [#78404](https://github.com/WordPress/gutenberg/pull/78404) · 6 comments · 0 reactions

## Summary

When a classic theme is active, the Site Editor (Appearance > Design) renders an inert preview of the site's front end via the `wp_site_preview` request—interactive elements are supposed to be non-functional. PR #75991, which introduced cross-origin isolation for client-side media features, broke this: the preview frame became interactive again and logged console errors in Chrome. This PR restores the intended inert behavior by sending the `Document-Isolation-Policy` header on the `wp_site_preview` response.

## Impact

- **Site owners / users:** Fixes a regression in the Site Editor's site preview for classic themes—clicking inside the preview no longer triggers front-end interactivity, and the associated browser console errors (seen in Chrome) are resolved. No action required.
- **Plugin & theme developers:** No API changes. This only affects the headers emitted on the internal `wp_site_preview` preview request used by the Site Editor when a classic theme is active.
- **Hosting & platform:** Be aware the `Document-Isolation-Policy` header is now also emitted on the `wp_site_preview` request (previously only used in the editor context), in addition to the headers added in #75991.

## Technical details

The change sends the `Document-Isolation-Policy` header on the `wp_site_preview` request, which serves the inert site preview shown in the Site Editor for classic themes. This is the same isolation header introduced for client-side media in #75991; applying it to the preview frame is what re-establishes the preview's expected non-interactive state (see the existing intent in `packages/edit-site/src/components/editor/site-preview.js`). Reviewer @adamsilverstein raised a concern that emitting this header on a preview could cause unexpected behavior—noting these headers are deliberately not sent on the front end and are only needed in the editor—and suggested possibly avoiding the headers on the outer preview frame instead, along with adding tests to catch the breakage.

The Core backport is tracked at WordPress/wordpress-develop#12004 (with `backport-changelog/7.1/12004.md`).

## Contribution

Opened and authored by @t-hamano (with assistance from Claude Code, reviewed by the contributor) as a follow-up to #75991, and merged in commit `4780ecd`. Reviewed by @adamsilverstein, who approved the code while flagging a concern about sending isolation headers on a preview frame and recommending regression tests; he also prepared the Core backport (wordpress-develop#12004) in draft pending merge. Props: t-hamano, adamsilverstein.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
