# #78556: Add dashboard Site Preview widget

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Type] Experimental`, `[Feature] Dashboard`
- **Merged:** [`891d5f7`](https://github.com/WordPress/gutenberg/commit/891d5f7c6132147369d345b6eccb0985ee31b870)
- **Discussion:** [#78556](https://github.com/WordPress/gutenberg/pull/78556) · 12 comments · 0 reactions

## Summary

PR #78556 introduces a new `core/site-preview` dashboard widget for the Gutenberg Dashboard (Beta) experience. The widget renders a scaled, read-only `<iframe>` preview of the site's homepage directly in wp-admin, with a "Customize" button that overlays the preview and redirects the user to the Site Editor on click. The feature is entirely gated behind the `gutenberg-dashboard-widgets` experiment flag and is part of the broader dashboard widgets initiative tracked in issue #77616.

## Impact

**Plugin & theme developers**
- No action required. The widget is experimental and only visible when the `gutenberg-dashboard-widgets` experiment is explicitly enabled.
- No new hooks, filters, or public APIs are introduced in this PR.

**Site owners / admins**
- Only affects sites running Gutenberg with the `gutenberg-dashboard-widgets` experiment turned on and using Dashboard (Beta). The widget must be manually added to the dashboard.

**Hosting & platform**
- No database changes, no new REST routes, no option changes observed in the available source material.

**No breaking changes or deprecations.**

## Technical details

A new widget registered under the name `core/site-preview` is added to the Gutenberg dashboard module system. The widget displays a scaled homepage preview — implemented as a scaled-down `<iframe>` pointed at the site's frontend URL — with a "Customize" button rendered on hover that navigates to the Site Editor.

No unified diff was included in the source material, so specific file paths and component internals cannot be confirmed. Based on the PR description and build output, the widget is compiled as part of the `build/modules/` bundle (the size diff reports 0 B net change, suggesting the widget module is new but not yet counted separately or is tree-shaken until the experiment is active).

**Design iteration during review:**
- An initial full-widget CSS scale/zoom animation on hover was removed after @jameskoster noted it implied the entire widget surface was a click target, which it is not.
- The "Customize" button received a CSS `box-shadow` elevation treatment to visually separate it from the site's own design, which can be visually busy.
- A spinner on the button was discussed; @simison noted redirects are near-instant in production but kept minimal feedback for slow connections.

A companion PR (#78563) was mentioned as a follow-up to add toolbar elevation styling.

## Contribution

Authored by @simison and merged at commit `891d5f7`. Key review input came from @jameskoster, who flagged the misleading hover zoom animation and suggested future enhancements (an address bar for switching the previewed URL, a "View" action to open the page in a new tab). @retrofox is also credited as a co-author. The hover animation removal was the only design change accepted before merge; the address bar and view-action ideas were deferred.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
