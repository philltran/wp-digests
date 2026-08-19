# Dev Chat summary: August 18, 2026

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Jb Audras
- **Published:** 2026-08-18
- **Tags:** `Devchat`, `General`, `Summaries`, `7.1`, `7.2`, `core`, `dev chat`
- **Link:** [https://make.wordpress.org/core/2026/08/18/summary-dev-chat-august-18-2026/](https://make.wordpress.org/core/2026/08/18/summary-dev-chat-august-18-2026/)
- **Usefulness:** 2/5

## Summary

Routine dev chat recap from the tail end of the WordPress 7.1 cycle: RC3 shipped August 12 and RC4 on August 17, with 7.1 slated to go live at WCUS on August 19. Trunk was branched on August 10, so `trunk` is now 7.2 development and the 7.1 branch is under double sign-off, with a commit freeze in effect until the Wednesday after release. The Accessibility Improvements dev note landed and the 7.1 Field Guide is published. Also announced: a 7.0.4 security release, the Contributor Toolkit 1.0, and a call for volunteers for 7.2.

## Impact

- **Site owners / platform teams:** WordPress 7.1 is releasing at WCUS on August 19 — final RC (RC4) is the last chance to test against a build that matches what ships. A **7.0.4 security release** was also announced; treat it as the usual patch-promptly item for sites still on the 7.0 branch.
- **Plugin & theme developers:** The 7.1 **Field Guide** is the compiled index of every dev note for this release — read it before certifying compatibility. The **Accessibility Improvements** dev note is the newest addition and is the one most likely to affect themes and admin UI markup.
- **Core contributors / committers:** `trunk` is 7.2 as of the August 10 branch. Anything intended for 7.1 goes to the `7.1` branch and needs **double sign-off**; a commit freeze is active until the Wednesday after release. A call for volunteers for the 7.2 release squad is open.
- **Everyone else:** No action required — this post carries no API or behavior change of its own.

Dev chat returns to its regular slot, **Wednesdays at 15:00 UTC**, starting the week after WCUS.

## Technical details

There is no code change in this item; the substance is release-branch state and process.

- **Branch topology:** As of August 10, `trunk` in the core Subversion/Git repo tracks 7.2. The `7.1` branch is frozen to release-critical work and every commit to it requires two committers signing off. A commit freeze applies until the Wednesday following release day.
- **Release artifacts:** 7.1 RC3 (Aug 12) and RC4 (Aug 17) are tagged; the 7.1 Field Guide is published, aggregating the cycle's dev notes. The Accessibility Improvements dev note was published during this window and is the last dev note called out before the Field Guide compilation.
- **Adjacent releases:** A 7.0.4 security release was announced separately; this summary links to it without enumerating the fixes.
- **Ecosystem tooling:** The WordPress Contributor Toolkit 1.0 was introduced. The summary does not describe its contents beyond the announcement.

One open item worth tracking outside core: [meta.trac #1181](https://meta.trac.wordpress.org/ticket/1181), which drew discussion among attendees at WCUS. @jeffpaul flagged it as a Meta rather than core matter and asked anyone with relevant experience to comment on the ticket.

## Contribution

A meeting recap rather than a change record — the only "discussion" is scheduling (return to Wednesdays 15:00 UTC, raised by @amykamala) and @jeffpaul's request for feedback on meta.trac #1181. @joedolson reported for the release squad that 7.1 is "in good shape." No design debate or rejected alternative is recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
