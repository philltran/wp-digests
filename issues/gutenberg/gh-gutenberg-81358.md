# #81358: UI: Add Spinner component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] UI`
- **Merged:** [`03811bd`](https://github.com/WordPress/gutenberg/commit/03811bd60582c4871666a7f8c467454cfbb50eaa)
- **Discussion:** [#81358](https://github.com/WordPress/gutenberg/pull/81358) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

A `Spinner` component has been added to `@wordpress/ui`, ported from `@wordpress/components`. It renders the same SVG track-plus-arc structure and 1.4s linear spin, but is styled with WPDS custom properties inside the wp-ui `@layer` cascade and drops the legacy component's hardcoded `margin: 5px 11px 0` and explicit `width`/`height` SVG attributes. The `@wordpress/components` Spinner is now flagged `not-recommended` in Storybook (a formal `@deprecated` pass is planned separately), and the `@wordpress/eslint-plugin` `use-recommended-components` rule now allows `Spinner` from `@wordpress/ui`. The port is a prerequisite for moving validated form controls off private `@wordpress/components` APIs.

## Impact

**Block/plugin & editor-extension developers**
- New public export: `import { Spinner } from '@wordpress/ui';`. No action required today — the `@wordpress/components` `Spinner` still works and has not been deprecated in code yet.
- If you do switch, expect **visual regressions from the missing default margin**. The legacy component ships `margin: 5px 11px 0` for inline form alignment; the wp-ui version has none. Add spacing yourself via `className`/`style`.
- The wp-ui version has no `width`/`height` attributes on the `<svg>`; size comes from CSS (`--wpds-dimension-size-2xs`, 16px) and is overridden with `style`/`className`, not props.
- Color is theme-token-driven rather than `currentColor`. This is explicitly flagged as unsettled and likely to change (see technical details).

**Gutenberg contributors**
- `use-recommended-components` will no longer flag `Spinner` imported from `@wordpress/ui`; the `@wordpress/components` Storybook entry now carries a `not-recommended` status with a pointer to the wp-ui docs.

**Site owners / hosting** — no action required; nothing ships to the editor UI as a behavior change.

## Technical details

The component lives at `packages/ui/src/spinner/spinner.tsx` and is re-exported through `packages/ui/src/spinner/index.ts` and `packages/ui/src/index.ts` (`export * from './spinner';`).

It is a `forwardRef< SVGSVGElement, ComponentProps< 'svg' > >` — there is no bespoke prop surface. All `<svg>` props are spread through, the ref lands on the `<svg>` element, and `className` is merged with the module class via `clsx`:

```tsx
<svg
	className={ clsx( styles.spinner, className ) }
	viewBox="0 0 100 100"
	role="presentation"
	focusable="false"
	{ ...props }
	ref={ ref }
>
	<circle className={ styles.track } cx="50" cy="50" r="50" vectorEffect="non-scaling-stroke" />
	<path className={ styles.indicator } d="m 50 0 a 50 50 0 0 1 50 50" vectorEffect="non-scaling-stroke" />
</svg>
```

Because `role="presentation"` is the default and props are spread after it, any accessible name is the caller's responsibility (`<Spinner aria-label="Loading" />`, as the unit test exercises).

`packages/ui/src/spinner/style.module.css` nests everything under `@layer wp-ui { @layer components { … } }` and consumes WPDS tokens:

- `.spinner` — `width`/`height: var(--wpds-dimension-size-2xs)`, `display: inline-block`
- `.track` — `stroke: var(--wpds-color-background-track-neutral)`
- `.indicator` — `stroke: var(--wpds-color-background-thumb-brand)`, `stroke-linecap: round`, `transform-origin: 50% 50%`, `animation: spinner-spin 1.4s linear infinite both`

Both strokes are `1.5px` with `fill: transparent`. A comment records that the animation is **intentionally not disabled under `prefers-reduced-motion`**, citing Note 4 of the WCAG 2.2 *Pause, Stop, Hide* understanding document.

Sizing before/after:

```jsx
// @wordpress/components
<Spinner style={ { width: '32px', height: '32px' } } /> // plus baked-in 5px 11px 0 margin

// @wordpress/ui — CSS-driven default, token-based override, no margin
<Spinner style={ {
	width: 'var(--wpds-dimension-size-lg)',
	height: 'var(--wpds-dimension-size-lg)',
} } />
```

Supporting changes: `Spinner` added to the `@wordpress/ui` `ALLOWLIST` in `packages/eslint-plugin/rules/use-recommended-components.js` (with a rule-tester case); the `manifest` Storybook tag moves from the components story to the new `Design System/Components/Spinner` story; the components story's `componentStatus.status` flips `recommended` → `not-recommended` with a `notes` pointer. Unit tests cover ref forwarding, prop pass-through, and `className` merging. Bundle impact was +6 B total.

## Contribution

Landed as part of the wp-ui migration tracked in #81230, following the recent SCSS-modules rework of the components `Spinner` (#80511). A review thread questioned the token choice for the track and arc colors; rather than block, @mirka merged with that unresolved to unblock @oandregal's validated-form-controls work, noting the team is "likely going to drop the themed track/thumb, and switch to a single `currentColor` indicator" in a follow-up. Props-bot credits @mirka, @juanfra, @oandregal, @aduth, and @ciampo. The formal deprecation of the `@wordpress/components` Spinner is deliberately deferred to a separate PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
