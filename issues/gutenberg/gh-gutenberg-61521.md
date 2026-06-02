# #61521: Upgrade to React 19

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tyxla
- **Labels:** `[Type] Enhancement`, `[Package] Element`, `[Package] Data`, `[Package] Server Side Render`, `[Package] Compose`, `[Package] Core data`, `[Package] Viewport`, `[Package] Plugins`, `[Package] Components`, `[Package] Blocks`, `[Package] Editor`, `[Package] Block library`, `[Package] Notices`, `[Package] Format library`, `[Package] Rich text`, `[Package] Block editor`, `[Package] Edit Post`, `[Package] Data Controls`, `[Package] Edit Widgets`, `[Package] Interface`, `[Package] Primitives`, `[Package] Icons`, `[Package] Keyboard Shortcuts`, `[Package] Edit Site`, `[Package] Preferences`, `[Package] Media Utils`, `[Package] Commands`, `[Package] Core commands`, `[Package] Patterns`, `No Core Sync Required`, `[Package] DataViews`, `[Package] Fields`, `[Package] Router`, `[Package] Theme`, `[Package] UI`
- **Merged:** [`414befb`](https://github.com/WordPress/gutenberg/commit/414befbd308a15fe1560fcb17884d714f2e31a59)
- **Discussion:** [#61521](https://github.com/WordPress/gutenberg/pull/61521) · 97 comments · 0 reactions

## Summary

Gutenberg upgrades its React dependency from 18.3.1 to 19.2.x across the entire monorepo, affecting every package that depends on `@wordpress/element` or React directly. The change lands as merged commit `414befb` and is labeled **No Core Sync Required**, meaning it ships in the Gutenberg plugin ahead of any WordPress core bundle update. The bundled `react-dom` vendor script grows by ~14.4 kB (+33%) as a direct result of the React 19 build. A companion Ariakit update (PR #60992) was required to silence React 19 `ref`-access warnings introduced by the new runtime.

## Impact

**Plugin & theme developers**
- Code running in the block editor now executes against React 19. React 19 makes `ref` a regular prop on function components, removing the need for `forwardRef`; any third-party component that reads `element.ref` directly (the pattern used by older Ariakit builds) will trigger a React 19 deprecation warning and may behave unexpectedly.
- `forwardRef` still works in React 19 but is no longer required for function components — new code can accept `ref` in the props object directly.
- Components relying on `ReactDOM.render` (removed in React 18, but any lingering usage) will hard-fail.
- **No breaking change to the public `@wordpress/element` API surface** — re-exported symbols (`createElement`, `Fragment`, hooks, etc.) remain available under the same names.

**Hosting & platform teams**
- This is a **Gutenberg plugin–only change** (labeled `No Core Sync Required`). Sites running the Gutenberg plugin will receive React 19; sites on WordPress core's bundled editor scripts are unaffected until a separate core sync lands.
- `build/scripts/vendors/react-dom.min.js` grows by ~14.4 kB; factor this into asset-budget monitoring.

**Site owners**
- No action required. The change is transparent at the content and front-end layer.

**Headless & REST consumers**
- No action required; this change is editor-side only.

## Technical details

`package-lock.json` bumps three dependencies at the monorepo root:

```diff
-"@types/react": "^18.3.27",
-"@types/react-dom": "^18.3.1",
+"@types/react": "19.2.13",
+"@types/react-dom": "19.2.3",

-"react": "18.3.1",
+"react": "19.2.x",
```

Build artifact size changes of note (minified, compressed):

| Artifact | Change |
|---|---|
| `build/scripts/vendors/react-dom.min.js` | +14.4 kB (+33.23%) |
| `build/scripts/vendors/react.min.js` | +323 B (+11.65%) |
| `build/scripts/element/index.min.js` | +437 B (+8.41%) |
| `build/scripts/vendors/react-jsx-runtime.min.js` | −154 B (−23.09%) |

The React 19 runtime removes the special treatment of the `ref` prop on React elements: `ref` is now passed through as a regular prop to function components, and accessing `someReactElement.ref` directly emits a deprecation warning. Ariakit's `packages/ariakit-react-core/src/utils/misc.ts` contained a fallback that read `element.ref`, which triggered those warnings in Gutenberg's unit-test suite. The fix was updating to a newer Ariakit release (tracked in PR #60992) that avoids the pattern.

All packages carrying a React dependency — including `@wordpress/components`, `@wordpress/block-editor`, `@wordpress/editor`, `@wordpress/compose`, `@wordpress/data`, `@wordpress/core-data`, `@wordpress/rich-text`, `@wordpress/notices`, and roughly two dozen others — are covered by this single monorepo version bump.

## Contribution

PR #61521 was opened and driven by **@tyxla** as an experimental container for React 19 compatibility work, with tracking issue #71336 coordinating the broader migration effort. **@Mamaduka** first noticed that unit tests produced `ref`-related warnings not seen during the React 18.3 upgrade. **@DaniGuardiola** and **@diegohaz** (Ariakit maintainer) were consulted; @diegohaz noted an earlier Ariakit fix for the `element.ref` pattern, but it proved insufficient for React 19's stricter enforcement. @tyxla resolved the warnings by updating Ariakit to its latest version, with the Ariakit bump progressed separately in PR #60992. The PR was originally marked "Do not merge" for experimentation but was ultimately merged at commit `414befb`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
