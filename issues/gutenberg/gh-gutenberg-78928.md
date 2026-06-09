# #78928: Media Editor: Replace the zoom slider with +/- buttons

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`195eff9`](https://github.com/WordPress/gutenberg/commit/195eff9c78b535ddc543dc355fb3359e2fea7da8)
- **Discussion:** [#78928](https://github.com/WordPress/gutenberg/pull/78928) · 10 comments · 0 reactions

## Summary

The WordPress block editor replaces the image zoom slider in the media editing interface with discrete +/- buttons. This change eliminates an unanchored percentage control whose reference frame shifted unpredictably during rotation and cropping, providing a consistent perceived zoom step regardless of current scale. All image transformation controls are now consolidated into a single toolbar for predictable spacing across viewports.

## Impact

- **Block Editor users**: Zoom is now controlled via explicit +/- buttons; pinch-to-zoom and scroll-wheel zoom remain functional for large adjustments. Undo history (Cmd/Ctrl+Z) continues to register each step.
- **Theme/plugin developers**: No public API changes or breaking modifications. If you override editor component styles targeting the former slider, update selectors to match the new button-based layout within `MediaEditorImageControls`.
- **Action required**: None for standard implementations.

## Technical details

- Replaces the previous zoom slider implementation with a pair of +/- buttons integrated directly into the shared `MediaEditorImageControls` component alongside rotate/flip and aspect-ratio actions.
- The consolidation resolves mismatched spacing in the mobile footer by treating all transformation controls as a single toolbar group.
- Layout behavior between 600–782px viewport widths is managed via scoped CSS overrides targeting the `ComplementaryArea` component’s hard-coded `medium` breakpoint, enforcing a column-to-600px transition without altering the underlying breakpoint logic or inline `100vw` positioning.
- State management remains tied to the existing undo manager, ensuring each +/- click is reversible via Cmd/Ctrl+Z.

## Contribution

- Merged by `@ramonjd` (co-authored with `@andrewserong`) at commit `195eff9`.
- Framed as a follow-up to the crop modal layout (#78896) and image-controls toolbar refactor (#78935).
- The PR focused on unifying the footer spacing and replacing the ambiguous percentage slider; follow-up work for refined responsive sidebar behavior and optional zoom animations was explicitly scoped out prior to merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
