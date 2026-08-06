# #80628: Fix: New route-based admin pages are empty when no JS

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @hbhalodia
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `Backported to WP Core`
- **Merged:** [`e16afac`](https://github.com/WordPress/gutenberg/commit/e16afacfd179e7904a208cb1da0369733a76c134)
- **Discussion:** [#80628](https://github.com/WordPress/gutenberg/pull/80628) · 12 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Route-based admin pages now correctly display a JavaScript-required notice when JS is disabled, instead of rendering as a blank screen. This fix scopes the template’s critical CSS to the `body.js` selector, ensuring legacy admin elements are hidden only when JavaScript is active. The change resolves an accessibility and usability regression introduced by the new single-page admin architecture.

## Impact

- **Site owners & administrators**: Will see a clear admin notice prompting them to enable JavaScript instead of a blank page.
- **Plugin & theme developers**: No code changes required. The fix applies automatically to the generated single-page admin templates used by route-based admin pages.
- **Hosting & platform teams**: No configuration or migration steps needed. The change is included in the Gutenberg plugin and has been backported to WordPress Core.

## Technical details

The change modifies `packages/wp-build/templates/page-wp-admin.php.template`. Previously, critical CSS selectors like `body`, `#wpcontent`, `#wpbody-content`, and `#wpfooter` applied globally, which inadvertently hid the no-JS admin notice when JavaScript was disabled. The diff prefixes these selectors with `body.js`, ensuring the hide-legacy-elements rules only execute when the `js` class is present on the `<body>` element.

Before:
```css
body { background: #fff; }
#wpbody-content > div:not(.boot-layout-container):not(#screen-meta) { display: none; }
```

After:
```css
body.js { background: #fff; }
body.js #wpbody-content > div:not(.boot-layout-container):not(#screen-meta) { display: none; }
```
This aligns the template’s styling behavior with WordPress’s standard `hide-if-js`/`show-if-js` pattern without altering the DOM structure or introducing new hooks.

## Contribution

Opened by @hbhalodia to address Trac ticket #65690 and Gutenberg issue #80626. During review, @t-hamano and @afercia debated whether the fix should inject `wp_admin_notice()` directly into the template or live in Core CSS, and discussed architectural concerns about Gutenberg overriding Core styles. The team agreed to scope the CSS to `body.js` within the Gutenberg template for a quick 7.1 release, deferring core-wide CSS adjustments to 7.2. The PR was merged and backported to Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
