# #73320: Global Styles: Add textShadow style support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `Needs Design Feedback`, `[Package] Block editor`, `Needs Dev Note`, `Global Styles`, `[Package] Style Engine`, `[Feature] Typography`
- **Merged:** [`e6b4c86`](https://github.com/WordPress/gutenberg/commit/e6b4c86ba6a5895df8808db35da3dbe042cea211)
- **Discussion:** [#73320](https://github.com/WordPress/gutenberg/pull/73320) · 14 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

Adds native `text-shadow` CSS property support to the Global Styles pipeline via `theme.json`. This enhancement allows theme authors to define text shadows globally or per-block through typography settings, fulfilling a core part of the broader #47904 feature request without yet exposing a UI control.

## Impact

- **Theme & plugin developers:** Can now define text shadows by adding `"textShadow": "1px 1px 2px red"` under `styles.typography` or block-specific paths in `theme.json`. No editor UI is provided with this release; the change strictly controls CSS output.
- **Site owners/editors:** Will see text shadows applied where supported by theme styles, but must rely on manual `theme.json` edits or plugin overrides since typography controls are not yet exposed.
- **Platform/Hosting:** Fully backward compatible. No migration steps, configuration changes, or code modifications are required.

## Technical details

- `lib/class-wp-theme-json-gutenberg.php`: Maps the CSS property `text-shadow` to the `typography.textShadow` configuration path and initializes it as a null default in the typography properties array.
- `packages/style-engine/src/styles/typography/index.ts`: Registers a new `textShadow` handler that passes the value through the existing style pipeline to generate matching CSS rules.
- `packages/style-engine/src/types.ts`: Updates the `Style` interface with `textShadow?: CSSProperties['textShadow']`, fixing TypeScript type safety for external consumers.
- Editor preview placeholders: `default-block-appender/content.scss` and `rich-text/content.scss` apply a `text-shadow: none;` reset to ensure placeholder text remains readable and does not inherit global typography styles.
- Schema & tests: `schemas/json/theme.json` is updated with the property definition, and both PHPUnit (`class-wp-theme-json-test.php`) and JS (`packages/style-engine/src/test/index.js`) suites include new assertions.

## Contribution

Opened by @t-hamano as a focused split from the larger typography feature tracker (#47904/#71419) to deliver style-only support first. Reviewers @aaronrobertshaw and @ramonjd requested TypeScript type additions, test coverage for the property, placeholder shadow resets, and documentation refresh. All requested changes were implemented prior to merge at `e6b4c86`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
