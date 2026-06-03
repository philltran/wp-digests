# #78479: UI Button: Fix disabled cursor style

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Bug`, `[Package] UI`
- **Merged:** [`24f2b61`](https://github.com/WordPress/gutenberg/commit/24f2b612216a85191d30e36cdaf682b0dc531af0)
- **Discussion:** [#78479](https://github.com/WordPress/gutenberg/pull/78479) · 4 comments · 0 reactions

## Summary

Fixes a CSS regression in the `@wordpress/ui` package where disabled `<Button>` components incorrectly displayed an interactive pointer cursor. The stylesheet now conditionally applies the `--wpds-cursor-control` variable only when the element lacks the `data-disabled` attribute, restoring visual parity between interaction state and user feedback.

## Impact

- **Plugin & Theme Developers:** No code changes required; the fix is applied to compiled CSS within the `@wordpress/ui` package.
- **Editor & Frontend Consumers:** Disabled buttons across all UI surfaces will now correctly render a non-interactive cursor, improving visual clarity without breaking existing CSS overrides or theme-specific link styling.

## Technical details

The change modifies `packages/ui/src/button/style.module.css`. Previously, the design-system cursor variable (`--wpds-cursor-control`) was applied unconditionally to all button states. The updated logic now gates this assignment behind a check for the presence of `data-disabled` on the element. Link-specific cursor overrides within the component are explicitly preserved and remain unaffected by the scoping change.

## Contribution

Opened and merged by `@mirka` with review from `@aduth`. The author noted the regression likely emerged when the default button cursor was shifted from `default` to `pointer`, exposing the missing disabled-state override. No alternative approaches were debated; the fix was accepted directly after confirming it met design system expectations for interactive states.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
