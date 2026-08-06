# Dev Chat Agenda – July 28, 2026

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Amy Kamala
- **Published:** 2026-07-28
- **Tags:** `Core`, `Devchat`, `General`, `7.1`, `agenda`, `core`, `dev chat`
- **Link:** [https://make.wordpress.org/core/2026/07/28/dev-chat-agenda-july-28-2026/](https://make.wordpress.org/core/2026/07/28/dev-chat-agenda-july-28-2026/)
- **Usefulness:** 3/5

## Summary

The WordPress 7.1 release cycle is advancing toward Beta 4, with the developers chat agenda highlighting several upcoming core capabilities. Key additions include client-side media processing, text shadow support in global styles, default infinite scroll for the media library, editable blocks inside the Custom HTML block, and new block supports for minimum width and background gradients. These changes will be documented in individual dev notes as they merge.

## Impact

- **Plugin & theme developers**: Will need to review upcoming dev notes for new block supports (`minWidth`, `background.gradient`), SVG icon registration APIs, and editor component updates. No immediate code changes are required until the features land in Beta 4/RC1.
- **Site owners & editors**: Will experience default infinite scroll in the media library and the ability to place editable blocks inside Custom HTML blocks.
- **Hosting & platform teams**: No immediate configuration changes required; monitor for React 19 experimental status in Gutenberg and standard 7.1 release timelines.
- **No action required** for existing codebases until the features are merged and documented in the 7.1 Field Guide.

## Technical details

This upstream item is a dev chat agenda and does not contain a unified diff or specific implementation details. The agenda references upcoming capabilities that will be grounded in concrete changes once merged: client-side media processing pipelines, global styles text shadow CSS generation, media library infinite scroll pagination logic, block binding/rendering updates for Custom HTML, SVG icon registration via editor components, and new block.json `supports` keys for `minWidth` and `background.gradient`. Developers should consult the linked dev notes for exact file paths, hook signatures, and schema updates.

## Contribution

The agenda coordinates the 7.1 beta cycle discussion, noting that Beta 3 shipped on July 22 and Beta 4 is scheduled for July 29. Core contributors are triaging remaining milestone tickets, with most slated for punt if not merged before RC1. The React 19 integration was explicitly deferred to experimental status in Gutenberg, reflecting a deliberate decision to isolate major framework upgrades until the release window stabilizes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
