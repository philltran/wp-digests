# #81357: Base Styles: Align `input-control` mixin to design system

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Type] Breaking Change`, `[Package] Components`, `[Package] Base styles`, `[Package] DataViews`
- **Merged:** [`be33031`](https://github.com/WordPress/gutenberg/commit/be330310c522d85a155a5bedcac796bce651ec33)
- **Discussion:** [#81357](https://github.com/WordPress/gutenberg/pull/81357) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The shared `input-control` SCSS mixin in `@wordpress/base-styles` now uses the design system's `outset-ring__focus()` mixin, WPDS border tokens, and a hover border treatment instead of the old border-plus-box-shadow focus style. This restyles `TextControl`, `TextareaControl`, `FormTokenField`, `ContentEditableControl`, `ComboboxControl`, their validated variants, and the DataViews filter search input. The deprecated `input-style__focus` mixin is removed, which is a breaking change for anyone who includes it from `@wordpress/base-styles`.

## Impact

**Plugin & theme developers (SCSS consumers of `@wordpress/base-styles`)**
- **Breaking:** `input-style__focus` is removed. Replace `@include input-style__focus;` with `@include outset-ring__focus;`. Builds that still reference the old mixin will fail at Sass compile time.
- `input-control` now renders a different focus ring (outset ring) and adds a hover border color. If you apply `input-control` and also define your own `:focus` `box-shadow` on the same selector, remove it to avoid a doubled focus ring (per the changelog note).
- `input-control` hover excludes `:disabled`, `[aria-disabled="true"]`, and `[type="checkbox"]`. Custom disabled states not expressed through those selectors (e.g. an `.is-disabled` wrapper) will pick up the hover border and may need an override; `FormTokenField` needed exactly this.
- Review any custom form controls that rely on the previous border/box-shadow focus look for visual regressions.

**Site owners / editor users**
- Visible restyle of text-like inputs in the editor and admin screens built on these components: focus rings and hover borders change. No functional change.

**Custom validated-control styling**
- Overrides that set `--wp-admin-theme-color` or `--wp-components-color-accent` to signal invalid state on these controls no longer drive the ring; the built-in styles now use `--focus-color` via `%red-focus-ring`.

No PHP, REST, or JS API changes.

## Technical details

**`packages/base-styles/_mixins.scss`**
- `input-style__neutral()` now uses `border: wpds.var("--wpds-border-width-xs") solid wpds.var("--wpds-color-stroke-interactive-neutral")` and `border-radius: wpds.var("--wpds-border-radius-sm")`. The transparent box-shadow and box-shadow transition are dropped.
- `input-style__focus()` is deleted.
- `input-control` gains a hover rule, `&:hover:not(:disabled, [aria-disabled="true"], [type="checkbox"])`, setting `border-color` to `--wpds-color-stroke-interactive-neutral-active`. A TODO notes the checkbox exclusion should go once `checkbox-control` hover is aligned.
- `input-control`'s `&:focus` now includes `outset-ring__focus()` and then resets `border-color` to the neutral stroke token and `box-shadow: none` to neutralize wp-admin `forms.css` focus styles.

```scss
// Before
&:focus { @include input-style__focus(); }

// After
&:focus {
  @include outset-ring__focus();
  border-color: wpds.var("--wpds-color-stroke-interactive-neutral");
  box-shadow: none;
}
```

**`@wordpress/components`**
- `combobox-control/style.scss` and `form-token-field/style.scss` swap `input-style__focus` for `outset-ring__focus` (on `:focus-within` and `.is-active` respectively). `FormTokenField`'s `.is-disabled` container gets a `&:hover` rule pinning `border-color` to `$components-color-gray-400`, with a TODO for WPDS disabled tokens.
- `text-control/style.scss` and `content-editable-control/style.module.scss` drop the explicit `border-color: $components-color-border` after `@include input-control`.
- `TextareaControl` (`index.tsx`) now applies a new CSS module class (`style.module.scss`, `.textarea { @include input-control; }`) via `clsx` alongside `components-textarea-control__input`. In `textarea-control-styles.ts`, the Emotion `inputStyleNeutral`/`inputStyleFocus`, font-family, mobile font-size, and `breakpoint('small')` rules are removed, leaving the mixin to supply them. The ESLint suppression count for that file drops from 2 to 1.
- Validated control styles (`validated-form-controls/style.scss`) replace `--wp-admin-theme-color` / `--wp-components-color-accent` overrides with `@extend %red-focus-ring` plus `border-color: $alert-red` for textarea and content-editable.

**`@wordpress/dataviews`**
- `validated-form-controls/style.scss`: `ComboboxControl` and `FormTokenField` invalid states use `@extend %red-focus-ring` (and `border-color: $alert-red`); the `ToggleGroupControl` rule swaps its `--focus-color` override for the same placeholder.
- `dataviews-filters/style.scss`: the filter search input's `:focus` inset box-shadow is removed, leaving the background change.

Changelog entries were added to `base-styles` (Breaking Changes and Enhancements), `components`, and `dataviews`.

## Contribution

Authored by @mirka as part of aligning legacy `@wordpress/components` form controls with `@wordpress/ui` (tracking issue #80412), and intended to merge alongside #80417, which did the same for `InputControl`, `SelectControl`, and `CustomSelectControl`. In review, a suggestion to warn external `base-styles` consumers more explicitly about focus-ring regressions (possibly with a dev note) was handled by adding a note to the changelog; a dev note was judged not applicable because the package isn't tied to the WP release schedule.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
