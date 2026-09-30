# #83463: RichText: Refactor useMarkPersistent, fix StrictMode mount mark

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`
- **Merged:** [`2ed980a`](https://github.com/WordPress/gutenberg/commit/2ed980a7c432274a18d0d8877ee39581eebee0d7)
- **Discussion:** [#83463](https://github.com/WordPress/gutenberg/pull/83463) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Refactors the internal `useMarkPersistent` hook in the block editor's RichText component to fix two bugs: (1) under React StrictMode (enabled by `SCRIPT_DEBUG`), the mount effect ran twice and the second run dispatched `__unstableMarkLastChangeAsPersistent`, creating a spurious undo step after loading a post; and (2) the falsy mount-detection check treated an empty string as "unmounted," so the first character typed into an empty field never started the 1-second undo debounce timer. The hook is restructured around a `getChangeType` classifier, `useDebounce`, and an injectable `onMarkPersistent` callback, with no change to the public API surface.

## Impact

- **Site owners / editors:** No visible change in production. The StrictMode undo-step bug only manifests with `SCRIPT_DEBUG=true`. The empty-field undo-timer fix is a subtle behavioral correction (first keystroke in an empty RichText field now correctly starts the 1 s debounce before creating an undo level).
- **Plugin & theme developers:** No action required. `useMarkPersistent` is an internal hook in `@wordpress/block-editor`; its signature change (adding the `onMarkPersistent` parameter) is not part of the public API. The `__unstableMarkLastChangeAsPersistent` dispatch is still used, just passed as a callback from `RichTextWrapper`.
- **Hosting & platform:** No configuration or migration changes. Bundle size delta is +144 B (0%).

## Technical details

The hook in `packages/block-editor/src/components/rich-text/use-mark-persistent.js` is restructured as follows:

**Before:**
```js
export function useMarkPersistent( { html, value } ) {
  const previousTextRef = useRef();
  const { __unstableMarkLastChangeAsPersistent } = useDispatch( blockEditorStore );
  // ...
  useLayoutEffect( () => {
    if ( ! previousTextRef.current ) {  // falsy = "mount"
      previousTextRef.current = value.text;
      return;
    }
    if ( previousTextRef.current !== value.text ) {
      const timeout = window.setTimeout( () => {
        __unstableMarkLastChangeAsPersistent();
      }, 1000 );
      previousTextRef.current = value.text;
      return () => window.clearTimeout( timeout );
    }
    __unstableMarkLastChangeAsPersistent();
  }, [ html, hasActiveFormats ] );
}
```

**After:**
```js
const TYPING_TIMEOUT = 1000;

function getChangeType( previous, next ) {
  if ( previous.html === next.html && previous.hasActiveFormats === next.hasActiveFormats )
    return 'none';
  return previous.text === next.text ? 'discrete' : 'typing';
}

export function useMarkPersistent( { html, value, onMarkPersistent } ) {
  const { text } = value;
  const hasActiveFormats = !! value.activeFormats?.length;
  const previousRef = useRef();
  const markPersistent = useEvent( onMarkPersistent );
  const markPersistentDebounced = useDebounce( markPersistent, TYPING_TIMEOUT );

  useLayoutEffect( () => {
    const previous = previousRef.current;
    const next = { html, text, hasActiveFormats };
    if ( ! previous ) { previousRef.current = next; return; }
    const changeType = getChangeType( previous, next );
    if ( changeType === 'none' ) return;
    previousRef.current = next;
    if ( changeType === 'typing' ) { markPersistentDebounced(); return; }
    markPersistentDebounced.cancel();
    markPersistent();
  }, [ html, text, hasActiveFormats, markPersistent, markPersistentDebounced ] );
}
```

Key changes:
- **Mount detection:** `previousRef` now stores the full `{ html, text, hasActiveFormats }` snapshot. The `getChangeType` classifier returns `'none'` when nothing changed, so a StrictMode re-run with identical state is a no-op instead of falling through to the discrete-mark path.
- **Empty-field fix:** The old `! previousTextRef.current` check was falsy for `''`, so an empty field was treated as unmounted. The new `! previous` check only fires when the ref is truly `undefined` (first render).
- **Debounce:** `useDebounce` from `@wordpress/compose` replaces the manual `setTimeout`/`clearTimeout` in the effect cleanup. `useEvent` wraps the callback so a new `onMarkPersistent` reference doesn't reset the debounce timer.
- **Call site** in `packages/block-editor/src/components/rich-text/index.jsx` (`RichTextWrapper`): the hook now receives `onMarkPersistent: __unstableMarkLastChangeAsPersistent` from `useDispatch(blockEditorStore)` instead of the hook dispatching internally.
- **New test file:** `packages/block-editor/src/components/rich-text/test/use-mark-persistent.jsdom.test.js` covers mount (normal + StrictMode), typing debounce, empty-field first character, html-lagging-behind-text, immediate mark on format change, callback swap without timer reset, and unmount cancellation.

## Contribution

Opened by @Mamaduka, closing long-standing issue #63167 and superseding the earlier attempt in #73378. Reviewer @tyxla raised questions about edge cases; @Mamaduka addressed the regression and test cleanup in a follow-up PR (#83531) and noted that two additional behavioral quirks pre-existed on trunk and were deferred. Co-authored with @ntsekouras and @stokesman. The PR notes Claude as an AI assistant. Merged as `2ed980a`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
