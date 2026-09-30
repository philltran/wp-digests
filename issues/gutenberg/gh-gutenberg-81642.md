# #81642: Editor: Surface debugging details in the ErrorBoundary

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Editor`
- **Merged:** [`427871b`](https://github.com/WordPress/gutenberg/commit/427871b0cf78b0273fc3bbe1593ad30e781fe28d)
- **Discussion:** [#81642](https://github.com/WordPress/gutenberg/pull/81642) · 41 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg editor's top-level `ErrorBoundary` now captures React's `errorInfo.componentStack` (previously discarded) and uses it in three places. The `editor.ErrorBoundary.errorLogged` action receives the error info as a second argument. The "Copy error" button now copies a Markdown report (message, stack, component stack, user agent) instead of the raw `error.stack`. The crash screen is also redesigned as a `Notice` plus collapsible details card, with the details panel shown when `SCRIPT_DEBUG` is on.

## Impact

- **Plugin & theme developers:** Callbacks on `editor.ErrorBoundary.errorLogged` now receive `( error, errorInfo )`. The change is additive, so existing single-argument callbacks keep working. Error-tracking integrations can now record `errorInfo.componentStack` to see which component threw, which helps when a plugin panel crashes the editor.
- **Developers with `SCRIPT_DEBUG` enabled:** A collapsible "Error details" panel in the crash screen shows the error message, stack, component stack, and user agent, so the console is not needed to identify the failing component.
- **Site owners / editors:** The crash screen looks different (a centered error `Notice` titled "The editor has crashed"). "Copy error" now produces a paste-ready Markdown bug report. "Copy contents" is still offered where `canCopyContent` is set (the post editor passes it).
- **Anything styling or testing the old markup:** The `.editor-error-boundary` styles changed (dark border, box-shadow and white background removed) and the fallback UI markup and copy strings changed. Custom CSS or e2e tests keyed on the old text, "The editor has encountered an unexpected error.", will need updating.
- No migration required.

## Technical details

Changes are in `packages/editor/src/components/error-boundary/index.js` and `style.scss`, plus the docs in `docs/reference-guides/filters/editor-filters.md`.

**Component stack capture**

```js
// before
componentDidCatch( error ) {
	doAction( 'editor.ErrorBoundary.errorLogged', error );
}

// after
componentDidCatch( error, errorInfo ) {
	this.setState( { componentStack: errorInfo?.componentStack } );
	doAction( 'editor.ErrorBoundary.errorLogged', error, errorInfo );
}
```

The component adds `componentStack: null` to its initial state.

**Report generation**
- `getErrorSections( error, componentStack )` builds the shared sections: the error name and message, `Stack`, `Component stack`, and `Environment` (user agent). The `Stack` and `Component stack` sections are only added when the stack data is present.
- `getErrorReport()` renders those sections as Markdown, under an `### Error report` heading with fenced blocks for the preformatted sections. This string is what "Copy error" copies.
- The same sections feed the in-UI `ErrorReport` component, so the copied text and the panel cannot drift apart.
- Report labels are deliberately untranslated.
- `getErrorName` and `getErrorMessage` handle non-`Error` throwables, falling back to `An unknown error occurred.`

**UI**
- The fallback now uses `Notice.Root intent="error"` with `Notice.Title`, `Notice.Description`, and `Notice.Actions` from `@wordpress/ui`.
- `CopyButton` now wraps `Notice.ActionButton`, replacing `@wordpress/components` `Button`. Its default variant is `outline`, and "Copy error" uses `solid`.
- `ErrorDetails` uses `CollapsibleCard` and is rendered only when `globalThis.SCRIPT_DEBUG` is truthy.
- A file-level `eslint-disable` for `@wordpress/use-recommended-components` is added, since the UI renders outside the editor's notice system.

**Styles**
- `.editor-error-boundary` moves to `--wpds-*` tokens: `--wpds-typography-font-family-body`, `--wpds-dimension-surface-width-lg` for max width, and `--wpds-dimension-padding-xl`.
- New `.editor-error-boundary__report` (max-height 420px, scrollable) and `.editor-error-boundary__report-section` (`pre-wrap`) classes are added.

**Docs**
- The `editor.ErrorBoundary.errorLogged` example now shows the `( error, errorInfo )` signature.

The PR's size report shows about +5.5 kB in `build/scripts/editor/index.min.js`.

## Contribution

Closes the long-standing issue #34482. @Mamaduka's first version used `EmptyState`, then asked for design help. @jasmussen reframed the screen as a "crash screen" rather than a block-style boundary and suggested a `Notice` followed by a `CollapsibleCard`, which the diff adopts. He also said the "Copy contents" action was optional. @ramonjd's design suggestions, including an error report, were folded in. @jsnajdr pointed out that `EmptyState` has no empty-state semantics and was constraining the width. He also asked for the raw `error.stack` alongside the component stack, questioned gating the details behind `SCRIPT_DEBUG`, and proposed a follow-up to give `BlockCrashBoundary` the same treatment. The merged diff adds the stack and keeps the `SCRIPT_DEBUG` gate. The author noted that Claude assisted with the PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
