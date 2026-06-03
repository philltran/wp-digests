# #78310: Blocks: Share window listeners across instances (block props, rich text, ...)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Performance`, `[Package] Compose`, `[Package] Components`, `[Package] Block library`, `[Package] Rich text`, `[Package] Block editor`, `[Package] Private APIs`
- **Merged:** [`463ee32`](https://github.com/WordPress/gutenberg/commit/463ee32e6126236f7600cb3e2eface79d9fd1688)
- **Discussion:** [#78310](https://github.com/WordPress/gutenberg/pull/78310) · 3 comments · 0 reactions

## Summary

The Block Editor now consolidates native event listeners across block instances by delegating them to shared document-level handlers instead of registering them per instance. This optimization reduces the total listener count from ~26k to ~2k on large documents, cutting down C++↔JS boundary crossings during editor mount and improving overall rendering performance in heavy content scenarios.

## Impact

- **Core package & internal developers**: Transparent refactor; no public API surface changes or deprecations.
- **Plugin & theme developers**: No code modifications required. The listener delegation shift is internal to block editor packages and does not alter how external blocks register handlers.
- **Hosting & platform teams**: Observe reduced DOM overhead and faster editor initialization in large posts. No configuration, migration, or compatibility steps needed.

## Technical details

- Introduces `subscribeSharedListener( target, eventType, callback, capture )` in `@wordpress/compose`. The helper lazily registers one native listener per `(target, eventType, phase)` tuple and fans out events to an internal subscriber `Set`.
- Replaces element-scoped listeners in `@wordpress/rich-text` (modules: `copy-handler`, `prevent-focus-capture`, `delete`, `format-boundaries`, `select-object`, `selection-change-compat`, `input-and-selection`) with shared handlers that bail out early via `! element.contains( event.target )`.
- Updates `@wordpress/block-editor` hooks (`use-focus-handler`, `use-is-hovered`, and rich-text wrappers like `enter`/`paste-handler`) to consume the shared delegation pattern.
- Migrates `@wordpress/block-library` entry handlers (`paragraph/use-enter`, `list-item/use-enter`, `list-item/use-space`, `button/use-enter`). Capture-phase attachment is preserved where needed to maintain correct event ordering against writing-flow `defaultPrevented` gates.
- Pattern shift example:
  ```diff
  // Before (per-instance element listener)
  useRefEffect(element => {
    element.addEventListener('keydown', handler);
    return () => element.removeEventListener('keydown', handler);
  });
  
  // After (shared delegation with containment check)
  useLayoutEffect(() => {
    const unsub = subscribeSharedListener(window, 'keydown', (e) => {
      if (!element.contains(e.target)) return;
      handler(e);
    }, capturePhase);
    return unsub;
  }, [element]);
  ```
- Net bundle impact: +661 B overall, with notable increases in `@wordpress/compose` (+491 B) and `@wordpress/rich-text` (+106 B).

## Contribution

Merged as commit [`463ee32`](https://github.com/WordPress/gutenberg/commit/463ee32e6126236f7600cb3e2eface79d9fd1688). Authored by @ellatrix with co-authors @jsnajdr and @ciampo (mciampini). The PR followed a straightforward internal review cycle with no public discussion or rejected alternatives documented in the thread. It shipped as a pure performance optimization aligned with React's synthetic event delegation model.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
