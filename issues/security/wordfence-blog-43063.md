# Inside a Malicious, Stealthy WordPress Must Use Plugin

- **Source:** Wordfence Blog
- **Type:** Blog post
- **Author:** unknown
- **Published:** 2026-09-22
- **Tags:** `Research`, `Threat Research`, `WordPress Security`, `WordPress Plugin Vulnerability News September 2026`, `WordPress Vulnerability News September 2026`
- **Link:** [https://www.wordfence.com/blog/2026/09/inside-a-malicious-stealthy-wordpress-must-use-plugin/](https://www.wordfence.com/blog/2026/09/inside-a-malicious-stealthy-wordpress-must-use-plugin/)
- **Usefulness:** 3/5

## Summary

Wordfence's Threat Intelligence Team documents a malware sample found during a site clean in mid-June that installs itself as a WordPress must-use plugin (mu-plugin) and disguises itself as an automated health-check and reporting tool. It uses several self-healing mechanisms to survive removal and uses Etherhiding, which places the attacker's server location in an Ethereum smart contract so the command channel is hard to take down. This is a threat-research writeup of an in-the-wild persistence technique, not a WordPress core or plugin change.

## Impact

**Site owners / agencies / hosts**
- Must-use plugins load on every request and do not appear as deactivatable in the standard Plugins screen, so a malicious file in `wp-content/mu-plugins/` is easy to miss during a routine audit.
- Samples were seen under more than 4,000 distinct filenames, commonly legitimate-looking ones such as the `advanced-cache.php` and `db.php` drop-ins and a theme's `functions.php`. Filenames are therefore unreliable indicators.
- The `Plugin Name`, `Author`, and `Plugin URI` header fields vary between samples, so header metadata is also unreliable for detection.
- Blocking or taking down a single C2 host is unlikely to be effective, since the endpoint is resolved through an Ethereum smart contract.
- Wordfence states a detection signature was released June 23rd 2026. Premium, Care, and Response customers received it immediately; free-version users received it after the standard 30-day delay.

**Plugin & theme developers**
- No API or behavior change. No action required in code.

**Incident responders**
- Because the malware has self-healing mechanisms, deleting the visible file alone may not be sufficient. The excerpt does not detail those mechanisms, so consult the original post for specifics before remediating.

## Technical details

The provided excerpt is truncated, so only the following is supported by the source:

- **Persistence location:** installed as a must-use plugin, which WordPress loads automatically on every request from the mu-plugins directory.
- **Disguise:** presented as an automated health check and reporting tool, with a plausible author name and a link to a code repository.
- **Filename/header variation:** more than 4,000 distinct filenames across detections, including `advanced-cache.php`, `db.php`, and a theme's `functions.php`. Plugin header fields (`Plugin Name`, `Author`, `Plugin URI`) differ between samples.
- **Command channel:** Etherhiding, meaning the attacker's server location is stored behind a smart contract on the Ethereum blockchain, making takedown of the command infrastructure difficult.
- **Design goal:** Wordfence describes nearly every component as aimed at either avoiding detection or surviving removal.

The specific self-healing implementation, contract interaction, payload behavior, and indicators of compromise are in the truncated portion of the post and are not reproduced here.

## Contribution

The record carries no discussion detail beyond the vendor's account that the sample was found during a site clean in mid-June and a signature shipped after QA on June 23rd 2026.

---

*Reported by Wordfence. Summarized from their published advisory — see the linked original for the full analysis. Wordfence is a trademark of Defiant, Inc.; this digest is not affiliated with or endorsed by them.*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
