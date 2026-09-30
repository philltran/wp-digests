# #83789: UI: Align component names with their semantics

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Breaking Change`, `[Tool] ESLint plugin`, `[Package] UI`, `[Package] Widget primitives`, `[Package] Widget Dashboard`
- **Merged:** [`9d5da7b`](https://github.com/WordPress/gutenberg/commit/9d5da7b263176e753662970d9710c78c6e3d7d48)
- **Discussion:** [#83789](https://github.com/WordPress/gutenberg/pull/83789) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/ui` renames `LinkButton` to `ButtonLink`. It also renames the `Dialog.CloseIcon`, `Drawer.CloseIcon` and `Notice.CloseIcon` sub-components to `Dialog.CloseIconButton`, `Drawer.CloseIconButton` and `Notice.CloseIconButton`. The new names put the component's kind last and its appearance or purpose first. `ButtonLink` is a link styled as a button, and `CloseIconButton` is an icon button for close or dismiss actions. Props and behavior are unchanged. The rename is marked as a breaking change in the experimental package.

## Impact

**Plugin and theme developers using `@wordpress/ui`**
- **Breaking:** `LinkButton` is no longer exported. Use `ButtonLink` instead.
- **Breaking:** `LinkButton.Icon` becomes `ButtonLink.Icon`.
- **Breaking:** the types `LinkButtonProps` and `LinkButtonIconProps` become `ButtonLinkProps` and `ButtonLinkIconProps`.
- **Breaking:** `Dialog.CloseIcon`, `Drawer.CloseIcon` and `Notice.CloseIcon` become `CloseIconButton` within each component. The changelog lists these as renames only.
- Update imports and JSX. No prop or behavior changes are needed.

**ESLint users**
- If you use the `use-recommended-components` rule from `@wordpress/eslint-plugin`, update to a version whose allowlist contains `ButtonLink`. Older versions only recognize `LinkButton`.

**Everyone else**
- No action required for site owners or REST consumers.

## Technical details

**Renames in `packages/ui`**
- The `src/link-button/` directory moves to `src/button-link/`. Files are renamed to match, for example `button-link.tsx` and `button-link.browser.test.tsx`.
- The CSS module class `.link-button` becomes `.button-link`.
- The Storybook title becomes `Components/@wordpress-ui/ButtonLink`, with id `design-system-components-buttonlink`.

**New files**
- `button-link/icon.tsx` adds a forwardRef `ButtonLinkIcon` that wraps `ButtonIcon`.
- `button-link/index.ts` attaches it with `Object.assign(_ButtonLink, { Icon: ButtonLinkIcon })` and sets `displayName = 'ButtonLink.Icon'`.
- `button-link/types.ts` defines `ButtonLinkProps`, which picks `variant`, `tone`, `size` and `children` from `ButtonProps` and omits them from `LinkProps`. It also re-exports `ButtonIconProps` as `ButtonLinkIconProps`.

**Implementation detail**
- `button-link.tsx` now declares the forwardRef component as `ForwardedButtonLink` with a named inner function `ButtonLink`, then exports it as `ButtonLink`.

**ESLint**
- `packages/eslint-plugin/rules/use-recommended-components.js` replaces `'LinkButton'` with `'ButtonLink'` in the `@wordpress/ui` allowlist.

**Docs and changelogs**
- `Button` JSDoc and the usage-guidelines MDX reference `ButtonLink`.
- `packages/ui/README.md` gains a "Component names" section.
- `packages/ui/CONTRIBUTING.md` gains a "Component naming" section. It covers the kind-last pattern, compound parts such as `Menu.LinkItem` and `Breadcrumb.LinkItem`, and compositions such as `ControlWithError` and `ChipWithRemove`.
- The UI and eslint-plugin changelogs record the change, including migration notes.

The diff was truncated, so the `CloseIconButton` file changes are not visible. The PR description and changelog say they follow the same pattern.

```tsx
// Before
import { LinkButton } from '@wordpress/ui';
<LinkButton href="/x"><LinkButton.Icon icon={ wordpress } />Go</LinkButton>

// After
import { ButtonLink } from '@wordpress/ui';
<ButtonLink href="/x"><ButtonLink.Icon icon={ wordpress } />Go</ButtonLink>
```

## Contribution

The rename follows the naming proposal in issue #82611. The PR notes it was implemented with Codex, and @mirka is credited alongside @ciampo. The record shows no design debate beyond that proposal.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
