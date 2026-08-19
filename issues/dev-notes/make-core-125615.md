# Dev Chat Agenda – August 18, 2026

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Jb Audras
- **Published:** 2026-08-17
- **Tags:** `Core`, `Devchat`, `General`, `7.1`, `7.2`, `agenda`, `core`, `dev chat`
- **Link:** [https://make.wordpress.org/core/2026/08/17/dev-chat-agenda-aug-18-2026/](https://make.wordpress.org/core/2026/08/17/dev-chat-agenda-aug-18-2026/)
- **Usefulness:** 3/5

## Summary

The August 18, 2026 Dev Chat agenda is primarily a WordPress 7.1 release-cycle status post: RC3 shipped August 12 and RC4 on August 17, with 7.1 slated to ship live at WCUS on August 19. Trunk branched on August 10, so trunk is now 7.2 development and the `7.1` branch requires double sign-off for commits. The agenda also flags the published Accessibility Improvements dev note and the WordPress 7.1 Field Guide, the 7.0.4 security release, a call for 7.2 release-squad volunteers, and the release of Contributor Toolkit 1.0. Dev Chat remains on Tuesdays at 15:00 UTC in #core for the rest of the 7.1 cycle.

## Impact

- **Site owners / platform teams:** 7.1 is at RC4 with a hard ship date of August 19 (live at WCUS). If you have not staged a 7.1 upgrade test, the window to catch regressions before general availability is effectively closed. Separately, the 7.0.4 security release is out — sites still on the 7.0 branch should be on 7.0.4.
- **Plugin & theme developers:** The 7.1 Field Guide is published and is the consolidated index of every 7.1 dev note; the Accessibility Improvements dev note is among them. Read the Field Guide rather than tracking individual dev notes.
- **Core contributors:** Trunk is open for 7.2 as of the August 10 branch. Anything intended for 7.1 must land on the `7.1` branch, which now requires **double sign-off** from committers. A call for 7.2 release-squad volunteers is open.
- **Anyone attending Dev Chat:** the meeting has moved to **Tuesdays 15:00 UTC** in the `#core` Slack channel for the duration of the 7.1 cycle — not the usual slot.
- No code changes are required by this post itself; it is a status and coordination announcement.

## Technical details

There is no code change here — this is a release-process agenda. The concrete process state it records:

- **Branch state:** 7.1 was branched from trunk on August 10, 2026. Trunk (`/trunk`) is now 7.2 development; the `/branches/7.1` line carries release-candidate work and is under the standard late-cycle double sign-off rule, meaning a second committer must approve before a commit lands there.
- **Release train:** 7.1 RC3 (August 12) → RC4 (August 17) → final on August 19 at WCUS. RC4 following RC3 by five days is the usual signal that late blockers were still landing.
- **Documentation artifacts:** the WordPress 7.1 Field Guide is published, which per the standard cycle compiles all 7.1 dev notes at the start of the RC phase. The Accessibility Improvements dev note is called out individually as published; the agenda does not enumerate its contents.
- **Adjacent releases:** 7.0.4 is a security release on the 7.0 branch. Contributor Toolkit 1.0 has been introduced as a separate ecosystem tool.

The agenda's Discussions section is empty (topics are solicited in the comments), and the meeting is structured as release status plus open floor, with tickets in the next major/maintenance milestone prioritized.

## Contribution

Routine curated agenda with no design debate on the record; discussion topics and ticket requests are solicited in the post's comments rather than pre-listed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
