# jQuery UI updated to 1.14.2 in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Aaron Jorbin
- **Published:** 2026-07-29
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`, `external libraries`
- **Link:** [https://make.wordpress.org/core/2026/07/29/jquery-ui-updated-to-1-14-2-in-wordpress-7-1/](https://make.wordpress.org/core/2026/07/29/jquery-ui-updated-to-1-14-2-in-wordpress-7-1/)
- **Usefulness:** 3/5

## Summary

WordPress 7.1 upgrades the bundled jQuery UI library from version 1.13.3 to 1.14.2. This update aligns with WordPress’s browser support policy by dropping legacy Internet Explorer and Edge support. To maintain backward compatibility with older jQuery UI patterns, the library now initializes with `jQuery.uiBackCompat` set to `true`.

## Impact

- **Plugin & theme developers**: Four internal jQuery UI symbols (`$.fn._form`, `$.ui.ie`, `$.ui.safeActiveElement`, `$.ui.safeBlur`) have been removed. Code that directly calls these will throw `undefined` errors in 7.1.
- **Site owners & hosting teams**: No direct action required, but third-party scripts relying on legacy jQuery UI helpers may fail.
- **Headless & REST consumers**: Unaffected.
- **Migration**: Audit custom scripts for the removed symbols and replace them with standard jQuery DOM methods or modern jQuery UI widget extensions.

## Technical details

The update replaces the `jquery-ui` asset bundle with version 1.14.2. During initialization, WordPress explicitly sets `jQuery.uiBackCompat = true` to preserve legacy widget behavior and API compatibility. The library diff removes four symbols from the jQuery UI namespace: `$.fn._form`, `$.ui.ie`, `$.ui.safeActiveElement`, and `$.ui.safeBlur`. Core does not reference these symbols, but any third-party code that does will encounter immediate breakage. Developers should migrate away from these legacy helpers:

```javascript
// Before (removed in 1.14.2)
$.ui.safeBlur( element );

// After (modern equivalent)
if ( element.blur ) {
    element.blur();
}
```

## Contribution

The update was tracked in Trac ticket #62757, with the asset replacement committed in #62747. Review was provided by @masteradhoc and @joedolson. The change followed a straightforward library bump to align with WordPress’s browser support policy, with no major design debates recorded in the dev notes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
