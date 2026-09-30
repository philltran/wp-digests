# #81365: wp-build: Render the no-JS fallback in the generated page templates

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Focus] Accessibility (a11y)`, `[Package] wp-build`
- **Merged:** [`25fc9c1`](https://github.com/WordPress/gutenberg/commit/25fc9c1204d7af2012ba607444106b32a9a4ce04)
- **Discussion:** [#81365](https://github.com/WordPress/gutenberg/pull/81365) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The two `wp-build` page templates (`page.php.template` and `page-wp-admin.php.template`) now render a `.wrap.hide-if-js` block containing the page heading and an error notice saying the screen requires JavaScript. Every generated wp-build admin page gets a no-JS fallback automatically, instead of each page (Connectors, Font Library) adding its own markup as in Trac #65690. The standalone `page.php.template` also needed a fix so `<body>` receives the `no-js` class and the script that swaps it for `js`.

## Impact

- **Plugin/theme developers using `@wordpress/wp-build`:** Regenerated pages pick up the fallback with no code changes. If you previously added your own no-JS heading/notice markup to a wp-build page, you will likely now see it duplicated and should remove yours.
- **Core / Gutenberg-bundled pages:** Connectors and Font Library (Appearance → Fonts, Settings → Connectors) show a heading and error notice instead of a blank screen when JavaScript is disabled.
- **Translators:** The new string is `This screen requires JavaScript. Enable JavaScript in your browser settings and reload the page.` It uses no text domain, deliberately, because these files are bundled into Core where the default domain applies.
- **Site owners / end users:** Only visible with JavaScript disabled. No action required.
- The accompanying `backport-changelog/7.2/12946.md` indicates a corresponding wordpress-develop PR for WordPress 7.2.

## Technical details

**`page-wp-admin.php.template`** (renders inside the regular wp-admin chrome): adds, immediately before the `#{{PAGE_SLUG}}-wp-admin-app` mount point:

```php
<div class="wrap hide-if-js">
	<h1 class="wp-heading-inline"><?php echo esc_html( get_admin_page_title() ); ?></h1>
	<?php
	wp_admin_notice(
		__( 'This screen requires JavaScript. Enable JavaScript in your browser settings and reload the page.' ),
		array( 'type' => 'error' )
	);
	?>
</div>
```

**`page.php.template`** (standalone full-page render that only ports the `<head>` portion of `admin-header.php`):

- `<body class="{{PAGE_SLUG}}">` becomes `<body class="{{PAGE_SLUG}} no-js">`.
- Adds the inline script from `admin-header.php`, `document.body.className = document.body.className.replace( 'no-js', 'js' );`, wrapped in `BEGIN/END see wp-admin/admin-header.php` comment markers. Previously `.hide-if-js` and `.hide-if-no-js` had no effect on these pages because `<body>` never had `no-js`/`js`.
- Adds the same `.wrap.hide-if-js` block (with `style="margin: 20px;"`) before the `#{{PAGE_SLUG}}-app` mount point.

Also adds a `packages/wp-build/CHANGELOG.md` entry and a backport-changelog file. No new hooks, filters, or REST changes.

## Contribution

This is a follow-up to #80628 and Trac #65690, which added the notice to the Connectors and Font Library pages individually; the work moves it into the templates to avoid duplication as more wp-build pages arrive. Core tracking is in Trac #65840 with a paired wordpress-develop PR (#12946). The PR description states it was authored with Claude Code and reviewed by the author, @t-hamano. The discussion contains only bot comments (props list and flaky e2e test reports unrelated to this change).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
