# #83480: Fix: Image lightbox: overlay becomes unusable (can't close) when it isn't a direct child of `<body>`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @hbhalodia
- **Labels:** `[Type] Bug`, `[Type] Regression`, `[Package] Block library`, `[Block] Image`, `[Package] E2E Tests`
- **Merged:** [`9277f8b`](https://github.com/WordPress/gutenberg/commit/9277f8b9d73f641deb45e05a89302a2d26532ade)
- **Discussion:** [#83480](https://github.com/WordPress/gutenberg/pull/83480) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Image block lightbox became impossible to close when its overlay was not a direct child of `<body>` — a regression introduced in WordPress 7.0. The `setInertElements()` function used the selector `body > :not(.wp-lightbox-overlay)`, which made the wrapper element containing the overlay inert, trapping the user inside an uncloseable lightbox. The fix replaces that flat selector with a walk-up-the-DOM approach that inerts only the overlay's siblings at each ancestor level, leaving the ancestors themselves untouched.

## Impact

- **Site owners / theme developers:** If your theme wraps the page in a container element (e.g. `<div id="site-wrap">`) so that `wp_footer()` output is not a direct child of `<body>`, the Image lightbox was uncloseable. This is fixed; no theme code change is required.
- **Plugin & theme developers:** No API change. The `inert` attribute is now applied to a narrower set of elements (siblings of the overlay at each level rather than all `body` children). If you were relying on the old behavior of *all* `body` children being inert while the lightbox is open, that is no longer the case — only the overlay's siblings at each ancestor level are affected.
- **No action required** for the vast majority of sites. The fix is transparent.

## Technical details

In `packages/block-library/src/image/view.js`, the `setInertElements()` action was rewritten.

**Before:**
```js
setInertElements() {
  document
    .querySelectorAll( 'body > :not(.wp-lightbox-overlay)' )
    .forEach( ( el ) => {
      if ( state.overlayEnabled ) {
        el.setAttribute( 'inert', '' );
      } else {
        el.removeAttribute( 'inert' );
      }
    } );
}
```

**After:**
```js
let inertElements = []; // module-level

setInertElements() {
  if ( ! state.overlayEnabled ) {
    inertElements.forEach( ( el ) => el.removeAttribute( 'inert' ) );
    inertElements = [];
    return;
  }
  const { ref } = getElement();
  let node = ref;
  while ( node && node !== document.body && node.parentElement ) {
    for ( const sibling of node.parentElement.children ) {
      if ( sibling !== node && ! sibling.hasAttribute( 'inert' ) ) {
        sibling.setAttribute( 'inert', '' );
        inertElements.push( sibling );
      }
    }
    node = node.parentElement;
  }
}
```

Key behavioral differences:
- The overlay's **ancestors** are no longer made inert (the old selector would inert the wrapper div that contained the overlay).
- Only **siblings** of the overlay at each level up to `<body>` are inered.
- A module-level `inertElements` array tracks exactly which elements received `inert`, so closing the lightbox removes the attribute only from those elements. Pre-existing `inert` attributes set by a theme are preserved.
- The `!sibling.hasAttribute('inert')` guard prevents double-inerting elements that were already inert before the lightbox opened.

A new E2E test plugin (`packages/e2e-tests/plugins/lightbox-overlay-wrapper.php`) hooks `wp_body_open` and `wp_footer` (priority 100) to wrap the page in `<div id="site-wrap">`, and a corresponding test in `test/e2e/specs/editor/blocks/image.spec.js` verifies that the wrapper is not inert, the page content sibling is inert while open, both are restored on close, and a separately-set `inert` on a `body` child survives the cycle.

## Contribution

Opened by @hbhalodia closing issue #83479, with review from @im3dabasia and @t-hamano. The author noted use of Claude Code (Opus 5.5) for the PR description, issue creation, and the fix itself, and stated they reviewed and tested the code. @t-hamano requested that a specific comment link be included in the merge commit description. The PR was approved and auto-merged.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
