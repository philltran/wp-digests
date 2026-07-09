# #79345: DataViews: Back richtext control with a @wordpress/components shell (alt to #78471)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Status] In Progress`, `[Package] Components`, `[Package] Rich text`, `[Type] Feature`, `[Package] Private APIs`, `[Package] DataViews`
- **Merged:** [`2a57cbe`](https://github.com/WordPress/gutenberg/commit/2a57cbee518160f93b13f37d7c5416485b709e35)
- **Discussion:** [#79345](https://github.com/WordPress/gutenberg/pull/79345) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/rich-text-control` package has been removed and its core UI shell migrated into `@wordpress/components` as a private API. This restructuring establishes a strict separation between a presentational form field (hosted in `components`) and the rich-text wiring logic, which is now owned by the consumer (typically `@wordpress/dataviews`). The change decouples the editable control from direct `@wordpress/rich-text` dependencies, enabling reuse across standalone form contexts while consolidating keyboard dispatch glue into the `rich-text` package.

## Impact

- **DataViews & Block Editor developers**: The `richtext` DataForm control must adopt the new assembly pattern, wiring `useRichText`, keyboard shortcuts, and format UI into the presentational shell's forwarded ref and `children` slot. Building custom rich-text controls outside DataViews will shift toward consuming this shell directly rather than relying on the legacy package.
- **Plugin & theme developers / headless consumers**: No immediate action required; the shell is exposed strictly as a private API and is not intended for external consumption yet.
- **Platform / internal tooling**: The standalone `@wordpress/rich-text-control` npm package is fully removed from the lockfile. Bundle size decreases slightly, and internal dependency graphs are simplified.

## Technical details

The diff replaces `packages/rich-text-control/` with a new `packages/components/src/rich-text-control/` structure:
- **Presentational shell**: `index.tsx` exports a `forwardRef`-wrapped `RichTextControl` component. It renders a `BaseControl` wrapper containing a `<div role="textbox" contentEditable>` and manages selection state via `useControlledValue` (`isSelected`, `defaultIsSelected`, `onSelectedChange`).
- **Focus tracking**: A document-level `focusout` listener targets `.popover-slot,.components-popover__fallback-container,[data-wp-compat-overlay-slot]` to defer deselection until focus leaves both the field and any open format popover. Selection is only toggled off when `blurDeselectTimeoutRef` completes.
- **Injection pattern**: The shell has zero `@wordpress/rich-text` dependency. Formatting behavior, shortcuts, and input rules are injected by the consumer via a forwarded ref and a `children` slot mounted only while the field is selected.
- **Private API exposure**: `RichTextControl` is registered in `packages/components/src/private-apis.ts`.
- **Test partitioning**: Shell tests (focus/blur, a11y, controlled selection) run in `packages/components/src/rich-text-control/test/index.tsx`. Integration tests for Enter/line-break handling and format input rules move to `packages/dataviews/richtext/test/control.js`.

## Contribution

Merged by @adamsilverstein as a stacked PR on #78471, this architectural shift emerged from review feedback by @ciampo and @mirka, who recommended hosting a presentational shell in `@wordpress/components` rather than shipping a dedicated package. @ntsekouras suggested relocating keydown/input dispatch helpers into `@wordpress/rich-text`, which was implemented to eliminate vendor duplication. The final diff addressed TypeScript conversion, restored Storybook stories under a `status-private` tag, and clarified the controlled/uncontrolled selection flow using `useControlledValue`. Co-authors include @adamsilverstein, @ntsekouras, @andrewserong, and @ciampo.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
