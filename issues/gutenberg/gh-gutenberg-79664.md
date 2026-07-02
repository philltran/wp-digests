# #79664: Media Editor: Use new Tabs component from the ui package, and its minimal variant

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`f09b052`](https://github.com/WordPress/gutenberg/commit/f09b05294f9583e3efa60422c08a1bfb8fc899b8)
- **Discussion:** [#79664](https://github.com/WordPress/gutenberg/pull/79664) · 4 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Replaces the legacy, internally unlocked `@wordpress/components` Tabs implementation in the Media Editor with the modern `@wordpress/ui` Tabs component and its `minimal` variant. This change fixes a visual alignment issue where tab labels did not line up with sidebar controls by removing horizontal padding from the minimal layout variant.

## Impact

- **Block & plugin developers:** No direct impact or breaking changes; the Media Editor is primarily consumed internally by WordPress core.
- **UI/Component maintainers:** Validates migration away from `@wordpress/components` private tabs API toward `@wordpress/ui`, demonstrating a lift-context pattern that bridges separate render regions.
- **Configuration/Migration:** None required. Internal component signatures were updated to use the new `Tabs.Root` context pattern and `value` props, aligning with core UI package standards.

## Technical details

- In `packages/media-editor/src/components/media-editor/index.tsx`, removes imports for `privateApis`, `unlock`, and `useContext`, replacing them with direct imports of `Tabs` from `@wordpress/ui`.
- Lifts `<Tabs.Root>` to wrap both `MediaEditorSidebar` and `InterfaceSkeleton` in `MediaEditorContent`, ensuring a shared context across the ComplementaryArea's header and body slots.
- Replaces `<Tabs.Context.Provider>` wrappers with native state management: `const [selectedTabId, setSelectedTabId] = useState<string>(); const activeTabId = selectedTabId ?? tabs[0]?.id;` persists selection across sidebar open/close cycles.
- Updates markup to `<Tabs.List variant="minimal">` to remove horizontal padding and flush labels left. Changes tab props from `tabId` to `value`, and replaces `<Tabs.TabPanel>` with `<Tabs.Panel tabIndex={-1}>`.
- Removes the corresponding ESLint suppression for `@wordpress/use-recommended-components` in `tools/eslint/suppressions.json`.

## Contribution

Opened by @andrewserong to address alignment feedback from @fcoveram, merged after brief review. Co-authored with @ramonjd. The discussion thread contained no major design debates or alternative approaches; the change was accepted as a straightforward modernization of the internal UI package usage.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
