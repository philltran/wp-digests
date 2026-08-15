# WordPress Core <= 7.0.2 - Unauthenticated Blind Server-Side Request Forgery

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-08-08
- **CVSS:** 5.8 (Medium)
- **CWE:** CWE-918: Server-Side Request Forgery (SSRF)
- **Affected:** WordPress (core) >= 4.7.0 and <= 4.7.33; >= 4.8.0 and <= 4.8.28; >= 4.9.0 and <= 4.9.29; >= 5.0.0 and <= 5.0.25; >= 5.1.0 and <= 5.1.22; >= 5.2.0 and <= 5.2.24; >= 5.3.0 and <= 5.3.21; >= 5.4.0 and <= 5.4.19; >= 5.5.0 and <= 5.5.18; >= 5.6.0 and <= 5.6.17; >= 5.7.0 and <= 5.7.15; >= 5.8.0 and <= 5.8.13; >= 5.9.0 and <= 5.9.13; >= 6.0.0 and <= 6.0.12; >= 6.1.0 and <= 6.1.10; >= 6.2.0 and <= 6.2.9; >= 6.3.0 and <= 6.3.8; >= 6.4.0 and <= 6.4.8; >= 6.5.0 and <= 6.5.8; >= 6.6.0 and <= 6.6.5; >= 6.7.0 and <= 6.7.5; >= 6.8.0 and <= 6.8.6; >= 6.9.0 and <= 6.9.5; >= 7.0.0 and <= 7.0.2
- **Patched in:** 4.7.34, 4.8.29, 4.9.30, 5.0.26, 5.1.23, 5.2.25, 5.3.22, 5.4.20, 5.5.19, 5.6.18, 5.7.16, 5.8.14, 5.9.14, 6.0.13, 6.1.11, 6.2.10, 6.3.9, 6.4.9, 6.5.9, 6.6.6, 6.7.6, 6.8.7, 6.9.6, 7.0.3
- **Researchers:** Andrew MacPherson
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/cddbb0f8-fc73-46d7-80af-edb69227084f](https://www.wordfence.com/threat-intel/vulnerabilities/id/cddbb0f8-fc73-46d7-80af-edb69227084f)
- **Usefulness:** 5/5

## Summary

Wordfence disclosed an unauthenticated blind SSRF in WordPress core affecting every release from 4.7.0 through 7.0.2. Core's outbound-request validation does not block all reserved and internal IP address ranges, so an unauthenticated attacker can induce the site to issue HTTP requests to attacker-chosen destinations — including hosts on the server's own private network — with the request originating from the web application rather than the attacker. The vulnerability is *blind*: the response body is not returned to the attacker, so the practical impact is enumeration of internal hosts, port/service discovery, and one-way interaction with internal endpoints that would otherwise be unreachable from the internet. Core shipped fixes across 24 maintenance branches (4.7.34 through 7.0.3).

## Impact

**Site owners / operators**
- Any WordPress install on an unpatched version is affected — this is core, not a plugin. Update to the patched release for your branch. Sites on the default minor-release auto-update path will pick this up automatically; verify rather than assume, particularly on installs where `WP_AUTO_UPDATE_CORE` has been disabled or `AUTOMATIC_UPDATER_DISABLED` is set.
- Sites still on long-abandoned branches (4.7.x–5.x) do have a patched release available — the backport reaches back to 4.7.

**Hosting & platform teams**
- Highest-value audience for this one. Blind SSRF from the web tier is primarily an internal-network reconnaissance primitive: cloud instance metadata endpoints, internal admin panels, container/orchestrator APIs, caches and databases bound to private addresses, and anything else reachable from the PHP worker but not from the internet.
- If you run WordPress at scale on shared infrastructure, patch level alone is not the whole mitigation — egress filtering on the PHP/worker tier and blocking metadata-service addresses at the network layer remain the durable controls.

**Plugin & theme developers**
- No API change and no code change required. If your plugin builds URLs from user-controllable input and passes them to the HTTP API, treat this as a reminder to validate destinations yourself rather than relying solely on core's blocklist.

**Headless / REST consumers**
- No REST schema or route change. No action required beyond updating core.

## Technical details

The advisory attributes the flaw to "insufficient validation of the destination address, as not all reserved and internal IP address ranges are blocked" — an incomplete-blocklist bug in the check that decides whether an outbound request target is external, not a missing check. It does not name the affected entry point, function, or file, and no patch diff accompanies the record, so the specific request-initiating vector is not public in this source.

For orientation, the core surface this lives on is the HTTP API's host validation: `wp_http_validate_url()` in `wp-includes/http.php` is what core uses to reject requests aimed at loopback and private-range addresses before `WP_Http::request()` dispatches them, and it is the path exercised by any core feature that fetches a user-supplied URL. Blocklist-style range checks of this shape are historically bypassed via ranges the list omits (for example less-common reserved blocks, alternate IP literal encodings, or IPv6 forms of a private IPv4 address). The advisory's wording — some reserved/internal ranges blocked, others not — is consistent with that class, but the precise omitted ranges are not disclosed here.

Defense-in-depth controls that exist in core independently of this fix, and that platform teams can layer on top of patching:

```php
// wp-config.php — deny all outbound HTTP from the HTTP API except an allowlist.
define( 'WP_HTTP_BLOCK_EXTERNAL', true );
define( 'WP_ACCESSIBLE_HOSTS', 'api.wordpress.org,*.wordpress.org,downloads.wordpress.org' );
```

The `http_request_host_is_external` filter also allows a mu-plugin to make the external/internal decision explicitly per host. Note that neither control is a substitute for updating core, and `WP_HTTP_BLOCK_EXTERNAL` will break plugins that legitimately call third-party APIs unless the allowlist is maintained.

No hooks, options, block APIs, REST routes, or database schema change as part of this fix.

## Contribution

The Wordfence record carries no development or discussion detail — only the researcher credit and the affected/patched version ranges. Nothing about the disclosure timeline, the core security team's triage, or the shape of the patch is present in this source.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
