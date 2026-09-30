# WP Digests

AI-generated summaries of **notable changes across the WordPress ecosystem** — core
(Trac + wordpress-develop), the block editor (Gutenberg), WooCommerce, the Make
WordPress dev blogs, and security releases and critical vulnerabilities — filtered by
impact and community interest.

Each notable change is one greppable Markdown file under [`issues/`](issues/), indexed
into per-track RSS feeds under [`feeds/`](feeds/). Updated periodically; each entry and
feed item carries its own date.

> ⚠️ **Summaries are AI-generated and may contain errors.** Every entry links to the
> authoritative upstream source — verify there before relying on a detail.

## Tracks & feeds

| Track | Covers | Feed |
|-------|--------|------|
| `gutenberg` | Block editor — the leading edge of future core features | `feeds/gutenberg.xml` |
| `wp-core` | Core engine changes (Trac tickets + wordpress-develop commits) | `feeds/wp-core.xml` |
| `dev-notes` | Dev notes, field guides, roadmap (the *why*) | `feeds/dev-notes.xml` |
| `woocommerce` | The largest plugin platform | `feeds/woocommerce.xml` |
| `security` | Core security releases, critical vulnerabilities & active-exploitation advisories | `feeds/security.xml` |
| `deprecations` | Cross-cutting: breaking changes & removed APIs | `feeds/deprecations.xml` |
| *all* | Everything | `feeds/all.xml` |

Browse the Markdown here on GitHub, or subscribe to a feed's raw URL.

> 🔒 **Security summaries are for awareness, not response.** Always verify against the
> linked advisory or release announcement before acting. Never patch, scope, or triage
> an incident from a summary alone: affected and patched versions, severity scores, and
> exploitation details must come from the source.

## Anatomy of an entry

Every entry is a Markdown file named `<source>-<id>.md`. It opens with a metadata list
(source, type, date, link, and a 1–5 usefulness grade, plus source-specific fields such
as labels, component, or milestone), followed by four sections: **Summary**, **Impact**,
**Technical details**, and **Contribution**.

The `security` track has three entry types:

| Filename | Source |
|----------|--------|
| `wp-news-security-<id>.md` | WordPress.org security-release announcements |
| `wordfence-blog-<id>.md` | Wordfence blog write-ups |
| `wfi-<uuid>.md` | Wordfence Intelligence vulnerability records |

Wordfence Intelligence entries add vulnerability metadata, taken from the advisory
record when it's present: **CVE**, **CVSS** (score and rating), **CWE**, **Affected**
(software and version range), **Patched in**, and **Researchers**. Their footer
reproduces the copyright and licence notice that the record requires.

## Inspired by

This project is directly inspired by Dries Buytaert's
[`drupal-digests`](https://github.com/dbuytaert/drupal-digests) — an AI-generated digest of
notable Drupal ecosystem changes. WP Digests applies the same idea to the WordPress ecosystem.

## Disclaimers & attribution

- **Not affiliated.** This is an independent, community project. It is **not affiliated
  with, sponsored by, or endorsed by** the WordPress project, the WordPress Foundation,
  or Automattic. "WordPress" and "WooCommerce" are trademarks of their respective owners
  and are used here only descriptively to refer to those projects.
- **Sources.** Summaries are derived from public, openly-licensed sources and each entry
  links back to the authoritative original (a Trac ticket, a GitHub pull request or
  commit, a Make WordPress or WordPress.org News post, a Wordfence blog post, or a
  Wordfence Intelligence record). Upstream content remains under its own license
  (WordPress core/Trac contributions are GPLv2-or-later; Gutenberg is GPL/MPL).
  Third-party security sources carry their own attribution and licence notices, which
  are reproduced in the footer of every entry derived from them.
- **AI-generated.** Entries are produced by an automated pipeline and may be inaccurate
  or incomplete.

## License

The digest text in this repository is licensed under
[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) — see also
[`NOTICE.md`](NOTICE.md). This license covers the *summaries*; it does not extend to the
upstream content they describe, which retains its own license.
