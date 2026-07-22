# #80287: Redirect editing events to extension handlers under editableRoot

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backport to Gutenberg RC`, `Backported to WP Core`
- **Merged:** [`3b8b102`](https://github.com/WordPress/gutenberg/commit/3b8b102d6b6a458560fb45d9027cfed66f6243e1)
- **Discussion:** [#80287](https://github.com/WordPress/gutenberg/pull/80287) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This PR fixes a regression introduced by the `editableRoot` writing flow refactor where keyboard, input, and composition event handlers attached via `wrapperProps` stopped firing. The block editor now bridges these specific editing events from the new editing host back to the block's registered handlers, preserving third-party extension functionality. This ensures compatibility for plugins and themes that rely on `editor.BlockListBlock` filters to attach input handlers under the new architecture.

## Impact

- **Plugin & theme developers:** Third-party code using `editor.BlockListBlock` filters to attach `onKeyDown`, `onInput`, `onBeforeInput`, or `composition*` handlers to `wrapperProps` will now work correctly under `editableRoot`. No code changes required.
- **Core developers:** No action required. This is a backward-compatibility shim; core block code does not depend on these handlers reaching the block element.
- **Site owners:** No direct impact.
- **Note:** `focus`/`blur` events are explicitly excluded from this bridge, as focus ownership now resides entirely with the host.

## Technical details

The diff implements a registration and dispatch bridge entirely within `@wordpress/block-editor` internals:
- `useRegisterBlockEventHandlers` (new hook in `packages/block-editor/src/components/block-list/use-block-props/use-register-block-event-handlers.js`) extracts supported `on*` props from `wrapperProps` and stores them in a new `eventHandlers` Map on the `BlockRefs` context.
- `useEditableRootEventHandlers` (new hook in `packages/block-editor/src/components/writing-flow/use-editable-root-event-handlers.js`) attaches native listeners for `keydown`, `keyup`, `beforeinput`, `input`, `compositionstart`, `compositionupdate`, and `compositionend` to the portal container/document element. When an event targets the host, it resolves the selected block and its ancestors via `getBlockParents`, then invokes the registered handlers innermost-first.
- Handlers receive a `createBlockSyntheticEvent` wrapper that mirrors React's `SyntheticEvent` interface (exposing `preventDefault`, `stopPropagation`, `nativeEvent`, and event-type-specific properties like `key` or `data`), ensuring `preventDefault` and `stopPropagation` behave identically to React's native event system.
- The `BlockRefs` context is extended with `eventHandlers: new Map()` to track per-block handler refs. No public APIs are modified.

## Contribution

Opened and merged by @ellatrix as a direct follow-up to #79105, addressing a regression identified during the `editableRoot` review process where third-party `wrapperProps` handlers were bypassed. The author implemented a lightweight compatibility shim that bridges events from the host to the block hierarchy without altering core block behavior. The PR was merged to meet the Gutenberg RC deadline and subsequently cherry-picked to the `wp/7.1` branch for inclusion in the next WordPress release. No alternative approaches were debated; the implementation focused strictly on restoring the expected bubbling behavior for extension handlers.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
