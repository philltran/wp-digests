# #80576: Notes: Register the inline note format at import time

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backported to WP Core`, `[Feature] Notes`
- **Merged:** [`2df157e`](https://github.com/WordPress/gutenberg/commit/2df157e7763d5ae6ab571f5604da7517891fe436)
- **Discussion:** [#80576](https://github.com/WordPress/gutenberg/pull/80576) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug where inline note markers rendered as `core/unknown` formatting after reloading the editor or switching to the code editor. The `core/note` rich-text format was previously registered inside a `useEffect` in the `NotesSidebar` component, which executed too late for eager HTML-to-rich-text conversion. The fix moves the `registerFormatType` call to an import-time side effect in `@wordpress/editor`, ensuring the format is available before any rich-text fields parse or render.

## Impact

- **Plugin & theme developers:** No action required. This is an internal editor bug fix with no public API changes.
- **Site owners & editors:** Inline note markers will no longer trigger "Clear Unknown Formatting" in the toolbar after reloading a post or switching editors.
- **Platform & hosting teams:** No configuration or migration needed. The change is backported to WordPress Core.

## Technical details

The diff moves `registerFormatType( NOTE_FORMAT_NAME, noteFormat )` from a lifecycle effect in `packages/editor/src/components/collab-sidebar/index.js` to a new file `packages/editor/src/hooks/note-format.js`, which is imported via `packages/editor/src/hooks/index.js`. Because `@wordpress/editor` declares a `sideEffects` allowlist that includes `hooks/` but excludes `src/components/**`, importing from `hooks/` guarantees bundlers won't tree-shake the registration call.

**Before:**
```js
useEffect( () => {
	registerFormatType( NOTE_FORMAT_NAME, noteFormat );
	return () => unregisterFormatType( NOTE_FORMAT_NAME );
}, [] );
```

**After:**
```js
// packages/editor/src/hooks/note-format.js
import { NOTE_FORMAT_NAME, noteFormat } from '../components/collab-sidebar/format';
registerFormatType( NOTE_FORMAT_NAME, noteFormat );
```

The `noteFormat` object in `packages/editor/src/components/collab-sidebar/format.js` also gains an `edit: () => null` property to satisfy the rich-text format API. Rich-text records are parsed eagerly during block parsing and initial field rendering; registering the format on import ensures `<mark class="wp-note">` markers are recognized before `useRichText` caches the parsed value.

## Contribution

Opened and merged by @Mamaduka, with review and testing from @adamsilverstein and @habiburdev. The author noted the bug was easy to miss because fresh markers work correctly, only failing on reload or editor switches. The team confirmed that moving registration to an import-time side effect aligns with how `@wordpress/format-library` handles core formats, and the `hooks/` path was chosen specifically to satisfy the package's `sideEffects` configuration. The PR was subsequently cherry-picked to the `wp/7.1` branch for inclusion in the next release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
