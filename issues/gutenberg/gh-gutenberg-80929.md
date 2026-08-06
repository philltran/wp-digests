# #80929: List View: Speed up opening by removing a forced style recalculation

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Feature] List View`, `[Package] Block editor`
- **Merged:** [`558ef39`](https://github.com/WordPress/gutenberg/commit/558ef399798fb90f76a422c8cab695dad3158c5d)
- **Discussion:** [#80929](https://github.com/WordPress/gutenberg/pull/80929) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block editor’s List View now opens faster by replacing an imperative `textContent` workaround in `AriaReferencedText` with a React `key`. The previous approach forced a full style recalculation on every commit, which significantly slowed down List View interactions in Firefox. The change eliminates that forced recalculation while preserving correct screen reader announcements.

## Impact

- **Block editor users**: Noticeably faster List View opening and navigation, particularly in Firefox.
- **Plugin & theme developers**: No action required. No public APIs, hooks, or block.json fields are affected.
- **Hosting & platform teams**: No configuration changes needed.

## Technical details

The change modifies `packages/block-editor/src/components/list-view/aria-referenced-text.js`. Previously, the component used `useRef` and `useEffect` to force `ref.current.textContent = ref.current.textContent` on every commit to work around a Firefox accessibility bug. The diff removes the ref and effect, instead applying `key={ children }` to the `<div hidden>` element. This instructs React to unmount and remount the node when the text changes, bypassing the in-place text update that triggered the style recalculation.

**Before:**
```jsx
const ref = useRef();
useEffect( () => {
    if ( ref.current ) {
        ref.current.textContent = ref.current.textContent;
    }
}, [ children ] );
return <div hidden { ...props } ref={ ref }>{ children }</div>;
```

**After:**
```jsx
return <div hidden { ...props } key={ children }>{ children }</div>;
```

## Contribution

The change was authored and merged by @Mamaduka, with co-authorship from @andrewserong. It was validated alongside related performance PRs in a combined run that reported nearly 50% faster List View metrics. No alternative approaches were debated in the thread; the solution was accepted after basic accessibility and keyboard testing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
