# #79810: Global Styles: Extract shared preset management from shadows and font sizes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Enhancement`, `Global Styles`
- **Merged:** [`ac9f753`](https://github.com/WordPress/gutenberg/commit/ac9f753c91837a0d6a4fd6d703757aa0078b625a)
- **Discussion:** [#79810](https://github.com/WordPress/gutenberg/pull/79810) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This PR extracts a shared preset management layer into `packages/global-styles-ui/src/presets/` to eliminate duplicated CRUD logic across the shadows and font sizes preset UIs. The change introduces a reusable `usePresets` hook, a `<PresetGroup>` list component, a `<PresetEditHeader>`, and unified confirmation dialogs for reset, delete, and rename operations. While the new components are not yet wired to the Global Styles UI in this commit, they establish a standardized pattern that will be adopted by upcoming spacing and border-radius preset migrations.

## Impact

- **Plugin & theme developers / UI contributors:** No action required. The extracted components are internal to the Global Styles UI and are not yet consumed by any screen.
- **Core maintainers & Gutenberg contributors:** Prepares the codebase for consistent preset management across typography, shadows, spacing, and border-radius. Follow-up PRs will migrate existing screens to use the new `usePresets` hook and `<PresetGroup>` component.
- **Breaking changes / deprecations:** None. Existing preset logic remains untouched until explicitly migrated.

## Technical details

The diff introduces a new `presets/` directory under `packages/global-styles-ui/src/` containing:
- `use-presets.ts`: A React hook that reads and mutates preset arrays at `${settingsPath}.${origin}` (e.g., `shadow.presets.custom`), explicitly fetching base values via `useSetting(..., undefined, 'base')` to support reset-to-theme functionality.
- `preset-group.tsx`: Renders a bordered, separated list of presets with an add button and a dropdown menu for bulk actions (e.g., reset).
- `preset-edit-header.tsx`: Provides a consistent header layout with a title, description, and a `Menu` popover for item-specific actions.
- `dialogs/confirm-delete-dialog.tsx`, `confirm-reset-dialog.tsx`, `rename-dialog.tsx`: Unified dialog components replacing near-identical implementations scattered across the codebase.
- `hooks.ts`: JSDoc updated to clarify that `readFrom` defaults to `"merged"`.
- `stories/index.story.jsx` & `storybook/main.ts`: Storybook configuration expanded to include `global-styles-ui` stories, with sample `shadow` and `fontSizes` theme data added for testing.
- `test/use-presets.js`: Unit tests verify path construction, base vs. custom origin handling, and empty array defaults.

No existing files were modified or removed; this is a pure addition. The intended consumption pattern mirrors `PaletteEdit`:
```ts
const { presets, basePresets, setPresets } = usePresets('shadow.presets', 'custom');
```
Follow-up PRs will replace the duplicated shadow and font size UI logic with `<PresetGroup>` and the new hook.

## Contribution

Opened by @juanfra to address duplication discovered while building spacing presets, with @Mamaduka co-authoring and reviewing. The approach mirrors the existing `PaletteEdit` pattern to maintain consistency. During review, @Mamaduka suggested wiring the new components to Storybook for easier testing; @juanfra implemented this and added sample theme values. Testing instructions were deferred to the follow-up migration PRs (#79811, #79812) to keep this refactor focused. The PR merged cleanly with no flaky test regressions tied to the changes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
