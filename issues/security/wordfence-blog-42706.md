# 40,000 WordPress Sites affected by Authentication Bypass Vulnerability in User Profile Builder WordPress Plugin

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-08-14
- **Tags:** `Research`, `Vulnerabilities`, `WordPress Security`
- **Link:** [https://www.wordfence.com/blog/2026/08/40000-wordpress-sites-affected-by-authentication-bypass-vulnerability-in-user-profile-builder-wordpress-plugin/](https://www.wordfence.com/blog/2026/08/40000-wordpress-sites-affected-by-authentication-bypass-vulnerability-in-user-profile-builder-wordpress-plugin/)
- **Usefulness:** 5/5

## Summary

User Profile Builder had a critical unauthenticated authentication bypass that let attackers log in as the user with ID 1, typically the site administrator, and take over the site. The flaw was only exploitable when the plugin's Automatically Log In setting was enabled. The issue was fixed in the latest patched release. Sites using the plugin should update immediately.

## Impact

- Site owners: update User Profile Builder to the latest patched version as soon as possible.
- Site owners: check whether the Automatically Log In setting is enabled, because the advisory identifies that setting as the exploit condition.
- Plugin and theme developers: no API, hook, or code change is described; the change is a security patch.
- Hosting and platform teams: prioritize patching or isolating sites running the vulnerable plugin.
- Headless and REST consumers: no REST route, block API, or configuration change is described.

## Technical details

The advisory describes an unauthenticated authentication bypass caused by type confusion in handling of the `username` parameter. An attacker can cause the plugin to authenticate as the user with ID 1, typically the administrator. The vulnerability is limited to installations where the Automatically Log In setting is enabled. The patched release corrects the flaw; no new hooks, REST routes, database changes, or configuration keys are described.

## Contribution

The record notes responsible disclosure through Wordfence's bug bounty program and a prompt developer patch; no design debate or alternative approach is documented.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
