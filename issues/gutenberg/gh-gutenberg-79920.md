# #79920: SandBox: Inject resize script into head to stop it leaking as text

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Block] HTML`
- **Merged:** [`508fdc8`](https://github.com/WordPress/gutenberg/commit/508fdc850dc468527f2a9545725fe78f362ff0b2)
- **Discussion:** [#79920](https://github.com/WordPress/gutenberg/pull/79920) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The SandBox component now injects its iframe resize helper script into the document <head> instead of the <body>. This prevents malformed user HTML—such as an unclosed attribute quote—from swallowing the inline script and leaking its source code as visible text in preview panes. To accommodate this structural shift, the script’s body-dependent setup is deferred until DOMContentLoaded, ensuring reliable execution regardless of parser state when content renders.

## Impact

['- **Plugin & theme developers**: No functional changes to <SandBox> behavior; if you extend or mock the component, ensure injected scripts still target <head> and defer DOM queries appropriately.', '- **Block authors (Custom HTML live preview)**: The live preview pane will no longer leak raw JavaScript source when rendering truncated or malformed markup.', '- **Platform/Hosting teams**: No configuration or migration required.', '- **Action required**: None for standard consumers. Verify custom SandBox wrappers only if you rely on its internal DOM structure for styling or testing.']

## Technical details

In `packages/components/src/sandbox/index.tsx`, the `observeAndResizeJS` IIFE was relocated from <body> to <head>. The initial top-level guard (`if ( ! MutationObserver || ! window.parent )`) was decoupled from body checks, and a new `connect()` function now handles all `document.body`-dependent mutations. This function runs synchronously if `document.body` is already available, otherwise it registers a `DOMContentLoaded` listener to defer viewport-style cleanup and the initial resize call.
The `buildSandBoxDocument` builder was exported and standardized across both isolated (`srcdoc`) and same-origin (`contentDocument.write`) paths. It now accepts an explicit `lang` parameter to avoid reading DOM globals during React render phases, preventing lint violations while keeping both code paths mathematically identical.
**Execution flow shift**:
```diff
// Before: Script injected after user HTML in body; runs immediately, risking parser state issues.
<body>
  <div dangerouslySetInnerHTML={{ html }} />
  <script>{`(observeAndResizeJS)()`}</script>
</body>

// After: Script injected in head before user HTML; body-dependent logic deferred to DOMContentLoaded.
<head>
  ...styles...
  <script>{`(observeAndResizeJS)()`}</script>
</head>
```
A test was added to `packages/components/src/sandbox/test/index.tsx` asserting that the resize script index is strictly less than the <body> tag index.

## Contribution

Opened and merged by @ramonJD with co-authorship from @ciampo. The change addresses a long-standing parsing edge case in the sandbox iframe that became highly visible after #73108 introduced live keystroke-by-keystroke preview to the Custom HTML block. To prevent future drift between the srcdoc and contentDocument.write code paths, the PR consolidates them into a single shared buildSandBoxDocument builder, with test coverage explicitly validating the head-vs-body ordering constraint.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
