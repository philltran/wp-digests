# #78898: Gallery Lightbox: Prevent image click from closing the dialog when navigation is enabled

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @i-am-chitti
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Gallery`
- **Merged:** [`472fcc8`](https://github.com/WordPress/gutenberg/commit/472fcc8702cae9577ec17464288454b55100599c)
- **Discussion:** [#78898](https://github.com/WordPress/gutenberg/pull/78898) · 12 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Clicking or tapping the enlarged image in the Image/Gallery lightbox no longer closes it. Dismissal now happens only through the Close button, the Escape key, or a click on the area around the image (the scrim). The original PR gated this on gallery navigation, but review feedback widened it to every lightbox, standalone images included.

## Impact

- **Site owners / editors:** Front-end behavior changes for all Image lightboxes. Visitors can tap the image to inspect it, or mis-tap while reaching for Previous/Next, without losing their place. Users who relied on tapping the image to dismiss it must now use Close, Escape, or the surrounding area.
- **Plugin & theme developers:**
  - Custom CSS or JS that assumes `.wp-lightbox-overlay` handles clicks on itself may break. The overlay no longer carries `data-wp-on--click="actions.hideLightbox"`.
  - Code that targets the overlay's click handler, or that relies on `cursor: zoom-out` over the image container, should be rechecked.
  - Tests that click the image to close the lightbox need to click the scrim or the Close button instead.
- **Headless / REST consumers:** Not affected.
- No migration is required. The markup is generated server-side, so existing content picks up the change without re-saving.

## Technical details

The diff touches three files in `packages/block-library`:

- **`src/image/index.php`** (lightbox overlay template):
  - Removes `data-wp-on--click="actions.hideLightbox"` from the `.wp-lightbox-overlay` root element.
  - Adds `data-wp-on--click="actions.hideLightbox"` to the `.wp-lightbox-close-button` button.
  - Adds the same directive to the `.scrim` div, which stays `aria-hidden`.
  - Clicks on the image container no longer bubble to a closing handler, because none exists on the overlay root any more.
- **`src/image/style.scss`:** Adds `cursor: default` to the lightbox image container rule, so the image no longer advertises a zoom-out affordance.
- **`CHANGELOG.md`:** Adds a Bug Fixes entry.

The PR description mentions a `handleImageContainerClick` action in `view.js` that calls `stopPropagation()` when `state.hasNavigation` is true, plus a `has-navigation` class toggle. The final diff contains none of that. It was dropped after review in favor of moving the click handler off the overlay root. `view.js` is not in the diff, and the diff includes no new tests.

```html
<!-- before -->
<div class="wp-lightbox-overlay zoom" data-wp-on--click="actions.hideLightbox" ...>
<!-- after -->
<div class="wp-lightbox-overlay zoom" ...>
  <button class="wp-lightbox-close-button" data-wp-on--click="actions.hideLightbox">
  <div class="scrim" data-wp-on--click="actions.hideLightbox" aria-hidden="true">
```

## Contribution

The PR started as a gallery-only fix (gated on `state.hasNavigation`) for issue #78869. @youknowriad questioned the inconsistency with standalone images, and @jasmussen agreed that whatever ships should be consistent and that "if it feels good, ship it". @youknowriad then proposed dropping the navigation gating and removing click-to-close on the image everywhere. @i-am-chitti reworked the PR accordingly. Swipe-to-dismiss was noted as a possible follow-up alongside #79114.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
