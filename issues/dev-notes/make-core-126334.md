# Dev Chat Agenda – September 30, 2026

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Amy Kamala
- **Published:** 2026-09-30
- **Tags:** `Core`, `Devchat`, `General`, `7.1`, `7.2`, `agenda`, `core`, `dev chat`
- **Link:** [https://make.wordpress.org/core/2026/09/30/dev-chat-agenda-sep-30-2026/](https://make.wordpress.org/core/2026/09/30/dev-chat-agenda-sep-30-2026/)
- **Usefulness:** 2/5

## Summary

This is the agenda for the September 30, 2026 WordPress core Dev Chat (15:00 UTC, #core on Make WordPress Slack). It lists five announcements: Gutenberg's JS unit and integration tests now use Vitest, a Trac MCP Server, WordPress Contributor Toolkit 1.2, a new Core release tools repo, and a shift from live-event releases to live-streamed releases. The one discussion item is the Secrets API (Trac #66187). @ericmann asked for broader conversation on the API approach before the 7.2 beta period, because a PR integrating 2FA with it has been opened.

## Impact

- **Plugin & theme developers:** No action required. The agenda is a meeting plan and carries no code changes. The Secrets API discussion (#66187) is the item to watch, because it could become a new core API. Anyone building credential or secret storage, or 2FA, should follow the ticket.
- **Gutenberg contributors:** The Vitest announcement suggests JS unit and integration test setup and commands may change. The agenda gives no details, so check the linked announcement post before updating local workflows or CI.
- **Core contributors:** The Trac MCP Server, Contributor Toolkit 1.2 and new release tools repo are contributor tooling. The agenda gives no details on any of them.
- **Site owners, hosts and REST consumers:** No action required.

## Technical details

The agenda has no code, diff, hooks or schema. It names only these items:

- **Announcements:** Gutenberg JS unit and integration tests moved to Vitest, a Trac MCP Server, WordPress Contributor Toolkit 1.2, a new Core release tools repo, and a shift from live event releases to live-streamed releases. The agenda gives titles only, with no implementation details.
- **Discussion:** Secrets API, Trac #66187. The agenda says a PR integrating 2FA has been opened. It does not describe the API's design, function names or storage model, and it says the aim is to establish a trajectory before the 7.2 beta period.
- **Open floor:** Tickets in the next major or maintenance release milestone will be prioritized. Participants are asked to post ticket and PR links in the comments and say whether they will attend live or take part async.

The post is tagged 7.1 and 7.2.

## Contribution

The post was written by Amy Kamala for the Dev Chat. The Secrets API topic was raised by @ericmann, who asked for a conversation on the API approach because a 2FA integration PR is already open. The record shows no other debate or alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
