# WordPress 7.0.3 release

- **Source:** WordPress News
- **Type:** Blog post
- **Author:** John Blackbourn
- **Published:** 2026-08-06
- **Tags:** `Releases`, `Security`, `minor-releases`, `releases`
- **Link:** [https://wordpress.org/news/2026/08/wordpress-7-0-3-release/](https://wordpress.org/news/2026/08/wordpress-7-0-3-release/)
- **Usefulness:** 5/5

## Summary

WordPress 7.0.3 is a security-only release fixing 12 separately reported vulnerabilities, headlined by a pre-authentication reflected XSS on the login screen that could be chained to PHP code execution (CVE-2026-64638 / GHSA-52p2-r8wf-jcrf). The rest are lower-privilege issues: four Contributor+ stored XSS vectors (the emoji settings element, the Post Content block, Quick Edit on large-user sites, the Post Date block), an Author+ CSS injection through a bypass of core's safe CSS attribute filter, a multisite privilege escalation letting an ordinary user create a new site where registration is open, and several information-disclosure and validation bypasses — password-protected post comments leaking through the Latest Comments block, post slug enumeration, notes disclosed in comment feeds, an email-confirmation flow bypass, and an SSRF in URL validation that permitted requests to link-local ranges. Because the login-screen bug is reachable without authentication and reportedly has a path to code execution, this is an update-now release rather than a routine minor. Fixes are being backported to every branch still eligible for security fixes (through 4.7), and 7.1 RC2 ships the applicable patches.

## Impact

**Site owners / operators**
- Update immediately. The login-screen XSS is pre-auth — no account, no user interaction beyond following a crafted URL — and is the one issue with a published advisory.
- Sites on background updates will pick 7.0.3 up automatically; anything with auto-updates disabled, or pinned via `WP_AUTO_UPDATE_CORE`/`automatic_updater_disabled`, needs a manual push.
- Multisite networks that allow user registration should treat the site-creation privilege escalation as an audit item: check for unexpected sites created before patching.

**Hosting & platform teams**
- Old-branch fleets are not fully covered yet — the 4.7-through-7.0 backports were explicitly still in progress at announcement and ship as they become ready, so a legacy site on, say, 5.9 may need a second patch window.
- The SSRF fix rejects requests to link-local ranges during URL validation. Any internal tooling that deliberately points core HTTP requests at link-local addresses (cloud instance metadata endpoints, link-local service discovery) will now be blocked. Audit before assuming no action.

**Plugin & theme developers**
- Expect the safe CSS attribute filter to be stricter after the Author+ CSS injection fix. If your product stores inline CSS authored by Author-or-lower users and round-trips it through core's KSES CSS filtering, re-test that your declarations still survive.
- Four of the fixes touch block output (Post Content, Post Date, Latest Comments) and Quick Edit. Re-run integration tests for blocks that wrap or re-render those cores, and for anything filtering the Latest Comments query — it now excludes comments on password-protected posts.

**Headless & REST consumers**
- The post slug enumeration fix and the Latest Comments disclosure fix both narrow what unauthenticated responses expose. Front-ends that were (knowingly or not) relying on enumerable slugs or on comment data from protected posts should be checked against 7.0.3 before deploy.

No deprecations or removed public APIs are announced in this release.

## Technical details

This is a release announcement, not a patch write-up: core has not published per-issue diffs (standard practice while backports to older branches are still landing), so the detail below is what the security team disclosed, grouped by class.

**Code execution / cross-site scripting**
- *Pre-auth reflected XSS on the login screen*, with a stated potential to lead to PHP code execution — the only issue with a public advisory (CVE-2026-64638 / GHSA-52p2-r8wf-jcrf). Reachable on `wp-login.php` without a session, which is what elevates it above the rest of the list.
- *Contributor+ stored XSS via the emoji settings element* — the emoji-detection settings script core prints in the document head.
- *Contributor+ stored XSS in the Post Content block* (`core/post-content`) and *in the Post Date block* (`core/post-date`).
- *Contributor+ stored XSS in Quick Edit*, specifically manifesting on sites with a large number of users — i.e. it depends on the user-list rendering path Quick Edit takes once the user count crosses core's threshold for switching from a `<select>` of authors to a different control.

**Privilege escalation**
- On multisite networks with user registration enabled, a user could create a new site — a capability check gap on the site-creation path rather than on registration itself.

**Information disclosure**
- The Latest Comments block (`core/latest-comments`) surfaced comments belonging to password-protected posts; the query is now constrained.
- Post slugs were enumerable.
- Notes were disclosed in comment feeds.

**Injection & validation bypasses**
- *Author+ CSS injection* through a bypass of core's safe CSS attribute filtering — the KSES-side allowlist that sanitizes `style` attribute content (`safecss_filter_attr()` in `wp-includes/kses.php`) and is extended by the `safe_style_css` filter. If your plugin adds properties through `safe_style_css`, retest them.
- *Email address confirmation flow bypass* — the confirm-link flow used when a user or admin changes an account email address.
- *SSRF in URL validation*: core's HTTP URL validation (the check behind `wp_safe_remote_*`, i.e. `wp_http_validate_url()`) permitted requests to link-local ranges (169.254.0.0/16 and IPv6 fe80::/10). Those are now rejected alongside the private ranges already blocked. The `http_request_host_is_external` / `http_request_reject_unsafe_urls` escape hatches remain the documented way to opt a specific integration back in.

No new hooks, `block.json` fields, REST schema changes, or database changes are announced.

## Contribution

The interesting operational detail is the release shape: rather than holding everything until all branches were ready, core shipped 7.0.3 first and let the backports to older eligible branches (down to 4.7) land as they became ready — so the older-branch fleet was knowingly left staggered behind the current branch for some window. The release also landed alongside 7.1 RC2, which carries the same applicable fixes, ten days before 7.1's launch at WordCamp US 2026 — a security release threaded into the final RC window of a major. Reporting was unusually broad for a single release, spanning an AI security vendor (pwn.ai) on the headline pre-auth bug, independent researchers, two members of the WordPress Security Team reporting issues themselves, and Anthropic on the safe-CSS-filter bypass; the SSRF was found by multiple independent reporters, which typically shortens the embargo window.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
