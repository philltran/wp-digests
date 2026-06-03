# #78656: Dashboard: URL bar in site preview widget

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Type] Experimental`, `[Feature] Dashboard`
- **Merged:** [`dc0f732`](https://github.com/WordPress/gutenberg/commit/dc0f7325aa227699f88348eabfc2fb9cb0b65786)
- **Discussion:** [#78656](https://github.com/WordPress/gutenberg/pull/78656) · 8 comments · 0 reactions

## Summary

The site preview widget in the experimental WordPress Dashboard gains a URL bar and optional favicon display in the widget's site bar. The PR also refines the hover overlay animation — replacing `@keyframes` with a simpler CSS `transition` — removes the drop-shadow from the preview button (now superseded by the overlay), and corrects the query parameter used in the preview iframe URL to suppress the admin bar.

## Impact

**Plugin & theme developers / dashboard extenders**
- No public API changes. This is a purely presentational change inside the experimental dashboard's site preview widget.
- No action required unless you are customizing the experimental dashboard UI directly.

**Site owners / administrators**
- Visible only when the experimental dashboard is opted into and the site preview widget is added. The widget now shows the site's URL (and favicon, if one is configured) in the chrome bar above the preview frame.

**All other audiences**
- No action required; the change is scoped entirely to the `[Feature] Dashboard` experimental flag.

## Technical details

The changes live inside the experimental dashboard feature, touching the site preview widget component (part of the `[Feature] Dashboard` work tracked in issue #77616, built on top of #78556).

**URL bar + favicon**
A site bar element is added to the top of the preview widget that renders the site's URL as visible text. When a site favicon is set, an `<img>` is conditionally rendered alongside the URL string. When no favicon is configured, only the URL text is shown (see the before/after screenshots in the PR).

**Animation refactor**
The hover overlay animation was previously driven by a `@keyframes` block. That block is removed and replaced with a plain CSS `transition` on the overlay element, making the interaction slower and more expressive while reducing the CSS footprint.

**Button shadow removal**
The `box-shadow` previously applied to the preview button is removed; the new hover overlay provides the necessary visual affordance.

**Admin bar URL param fix**
The iframe `src` used to preview the site was passing an incorrect query parameter to hide the WordPress admin bar. This is corrected to use the right param, ensuring the preview renders without the admin bar as intended.

No new hooks, filters, REST endpoints, block.json fields, or database changes are introduced.

## Contribution

Authored and opened by @simison around 26 May 2026 as a direct follow-up to #78556 and part of the broader experimental dashboard effort in issue #77616. @retrofox is credited as co-author. In the PR discussion, @simison noted that a more expressive URL bar style (matching the Settings page chrome) was considered but deliberately rejected as too prominent for the widget context, opting for the simpler presentation that shipped. Merged at commit `dc0f732` with 8 comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
