# #78466: Tooltip migration: editor + edit-post + edit-site consumers (2/5)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Edit Post`, `[Package] Edit Site`
- **Merged:** [`3db33d4`](https://github.com/WordPress/gutenberg/commit/3db33d485eb141d4fcbcc3e98fbda082de226469)
- **Discussion:** [#78466](https://github.com/WordPress/gutenberg/pull/78466) · 3 comments · 0 reactions

## Summary

Seven `Tooltip` call sites across `@wordpress/editor`, `@wordpress/edit-post`, and `@wordpress/edit-site` are migrated from the legacy `Tooltip` in `@wordpress/components` (Ariakit-backed) to the new compositional `Tooltip` in `@wordpress/ui` (base-ui-backed). Shell-level `<Tooltip.Provider>` components are mounted inside both the post editor and site editor layout trees, enabling coordinated hover-delay across an entire editor shell — once the first tooltip in a group has been shown, subsequent siblings open instantly. This is part 2 of a 5-PR series; no visual change is intended at any migrated site.

## Impact

**Plugin and theme developers**
- No action required. The `Tooltip` export from `@wordpress/components` is untouched; call sites *inside* `@wordpress/components` (e.g. `Button`'s internal tooltip, `TooltipInternalContext`) remain on the legacy implementation and are tracked as a separate follow-up.
- Code importing `Tooltip` directly from `@wordpress/ui` will need `// eslint-disable-next-line @wordpress/use-recommended-components` until the allow-list is flipped after the series completes.

**React Native / Gutenberg mobile**
- `packages/edit-post/src/components/layout/index.native.js` is intentionally left on the legacy import — it uses `<WCTooltip.Slot>`, the SlotFill pattern of the old implementation, which has no equivalent in the new compositional API.

**Accessibility**
- Two non-interactive triggers — the `<time>` element in `collab-sidebar/note-byline` and the `<div role="img">` in `collaborators-presence/avatar` — remain not keyboard-reachable. Both are explicitly flagged for a11y follow-up and are not a regression from the legacy behaviour.

**Bundle size**
- `build/scripts/edit-post/index.min.js`: +32.5 kB (+167%) — first pull of the `@wordpress/ui` base-ui runtime into this entry point.
- `build/scripts/edit-site/index.min.js`: +29.2 kB (+11%).
- `build/scripts/editor/index.min.js`: +29.6 kB (+6.85%).

**Site owners / headless / REST consumers**
- No action required.

## Technical details

**Import change at each migrated file**

```js
// Before
import { Tooltip } from '@wordpress/components';

// After
// eslint-disable-next-line @wordpress/use-recommended-components
import { Tooltip } from '@wordpress/ui';
```

The `eslint-disable` directive is required at every migrated site because `@wordpress/ui`'s `Tooltip` is not yet on the `@wordpress/use-recommended-components` allow-list. The recommendation will be flipped in a single commit after the full series lands.

**Migrated files**
- `packages/editor/src/components/collab-sidebar/note-byline`
- `packages/editor/src/components/collaborators-presence/avatar`
- `packages/editor/src/components/post-revisions-preview/diff-markers`
- `packages/editor/src/components/resizable-editor/resize-handle`
- `packages/editor/src/components/template-actions-panel/block-theme-content`
- `packages/edit-post/src/components/layout` (resize separator)
- `packages/edit-site/src/components/resizable-frame`

**Shell-level `<Tooltip.Provider>`**

Mounted inside the existing `<SlotFillProvider>` in both:
- `packages/edit-post/src/components/layout/index.js`
- `packages/edit-site/src/components/layout/index.js`

The provider coordinates hover-delay for all descendant tooltips in the editor shell: after the first tooltip in a session is shown, siblings open with zero delay. PR 5 of the series will add the equivalent to `@wordpress/boot`.

**Test rewrite — `collaborators-presence/avatar`**

The legacy Ariakit `TooltipAnchor` silently added `tabindex="0"` to non-interactive trigger elements; base-ui's `Tooltip.Trigger` does not. `packages/editor/src/components/collaborators-presence/avatar/test/index.tsx` was rewritten to drop that reliance. Tests now wrap the tree in `<Tooltip.Provider delay={ 0 }>` and assert tooltip visibility by hovering and awaiting:

```ts
// Before (selector-internal hack)
getByText( name, { selector: 'div *' } );

// After
await screen.findByText( name );
```

**Codemod**

Mechanical changes were applied via the jscodeshift codemod landed in PR #78411:

```sh
npx jscodeshift -t tools/codemods/tooltip-components-to-ui.js \
    --extensions=js,jsx,ts,tsx --parser=tsx \
    packages/editor packages/edit-post packages/edit-site
```

Import ordering and one `jsx-a11y` disable directive the codemod could not preserve were completed by hand.

## Contribution

Authored by @ciampo with AI assistance (Cursor/Claude); all changes were human-reviewed and exercised locally (Jest, lint, Prettier) before push. The PR follows #78095, which introduced the `@wordpress/ui` compositional `Tooltip` API, and #78411 (part 1 of the series), which migrated `@wordpress/block-editor` and `@wordpress/block-directory` and landed the jscodeshift codemod. Discussion on the PR consists of the automated bundle-size report and inline screenshot comments from the author; no alternative approaches were debated.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
