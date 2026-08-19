# #81785: Modal: Stop Escape key propagation on dismiss

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Components`
- **Merged:** [`da621f2`](https://github.com/WordPress/gutenberg/commit/da621f2e6a8757a14c7fe4718b4c8933f2e60c76)
- **Discussion:** [#81785](https://github.com/WordPress/gutenberg/pull/81785) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `Modal` component in `@wordpress/components` now stops Escape key propagation when it handles the key to dismiss itself. This prevents a single Escape press from also closing an underlying overlay when a modal is opened without closing its originating overlay. The fix also avoids removing the element that the modal should restore focus to. It is a targeted keyboard-behavior bug fix for stacked overlay UIs.

## Impact

- **Plugin & theme developers:** No action required unless you rely on an Escape key press bubbling out of `Modal` to close an underlying overlay or trigger a document-level `keydown` handler. In stacked overlay UIs, one Escape now closes only the topmost `Modal`.
- **Site owners:** No direct action required; admin UIs using nested modals may behave more predictably with keyboard dismissal.
- **Headless & REST consumers:** Not affected.

## Technical details

In `packages/components/src/modal/index.tsx`, the `UnforwardedModal` Escape branch now calls `event.stopPropagation()` after `event.preventDefault()` and before `closeModal().then( () => onRequestClose( event ) )`.

Before:

```js
event.preventDefault();
closeModal().then( () => onRequestClose( event ) );
```

After:

```js
event.preventDefault();
event.stopPropagation();
closeModal().then( () => onRequestClose( event ) );
```

A regression test in `packages/components/src/modal/test/index.tsx` renders `Modal`, adds a native `document.addEventListener( 'keydown', documentKeyDownHandler )` listener, presses `[Escape]` with `user.keyboard`, and expects `documentKeyDownHandler` not to be called. A `CHANGELOG.md` entry was added under `@wordpress/components` bug fixes.

## Contribution

The change was a follow-up to #79560 and closed #81774. @Mamaduka suggested it might also solve #81775, then reported difficulty reproducing the issue on trunk; the author clarified that reproduction involved #81564. The PR notes Codex was used to investigate, implement, and test the change, with the author reviewing the result.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
