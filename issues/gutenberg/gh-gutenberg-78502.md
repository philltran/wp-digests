# #78502: Dashboard: show ghost widgets visually & allow easy removal

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Type] Experimental`, `[Feature] Dashboard`
- **Merged:** [`1e95e7a`](https://github.com/WordPress/gutenberg/commit/1e95e7adec84e2a521247ae52038d1dfe4309ed1)
- **Discussion:** [#78502](https://github.com/WordPress/gutenberg/pull/78502) · 10 comments · 0 reactions

## Summary

When the WordPress Dashboard layout references a widget whose plugin has since been deactivated or removed, the slot previously rendered `null` — producing an invisible gap with no explanation. This PR introduces a visible "missing widget" placeholder that occupies the slot, communicates that the widget is unavailable, and provides a one-click button to remove the stale layout entry. The fix is scoped to the experimental Dashboard feature (tracked under issue #77616).

## Impact

**Plugin & theme developers**
- No action required. The fallback rendering is handled internally by the dashboard widget renderer; registered widgets are unaffected.
- Developers who register dashboard widgets should be aware that if their plugin is deactivated, any saved layout references will now surface visibly to admins rather than silently consuming grid space.

**Site owners / admins**
- Stale widget references left behind by deactivated plugins will now appear as labeled placeholders with a removal action, rather than causing unexplained blank areas in the dashboard.

**Hosting & platform teams**
- No configuration or migration required. This is a purely presentational change within the experimental Dashboard UI.

> **Note:** This feature is marked `[Type] Experimental`. It is part of the in-progress Dashboard redesign and is not yet in stable WordPress core.

## Technical details

The change is client-side React within Gutenberg's experimental Dashboard module. Previously, rendering a widget whose ID is not present in the registered widget registry produced `null`, leaving an invisible gap in the grid layout. The PR adds a fallback component — a "missing widget" placeholder — that is rendered instead when a layout entry's widget ID cannot be resolved.

A loading-state guard was required: during the async resolution of widget registrations, the missing-widget state must not trigger until the widget list has fully loaded, otherwise the placeholder appears transiently for all widgets on page load (a race condition @simison identified during development and addressed before merge).

The placeholder is rendered in both regular dashboard mode and customize/edit mode, and exposes a removal action that deletes the orphaned entry from the saved layout config.

No diff was provided with this source material; specific component names, file paths, store selectors, or dispatch actions cannot be cited without risk of invention.

## Contribution

Opened and authored by @simison as part of the larger Dashboard redesign tracked in issue #77616. @retrofox and @jameskoster contributed and are credited as co-authors. During review, @simison self-identified a loading-state regression where the placeholder was appearing prematurely before widget data resolved, and addressed it prior to merge. Merged as commit `1e95e7a` with 10 comments on the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
