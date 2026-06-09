# #78815: Dashboard: event widget iteration

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Type] Experimental`, `[Feature] Dashboard`
- **Merged:** [`67ca220`](https://github.com/WordPress/gutenberg/commit/67ca2203136096256c3c8925abd5e6afded03f67)
- **Discussion:** [#78815](https://github.com/WordPress/gutenberg/pull/78815) · 7 comments · 0 reactions

## Summary

The experimental Dashboard’s Events widget now persists location selection as a user attribute, restructures its layout for improved narrow-viewport resilience, and refines visual states to distinguish between viewing and modifying locations. These iterations streamline the discovery flow while centralizing location management in the dedicated settings panel.

## Impact

- **Experimental dashboard users**: Location choice is now persisted across page loads via widget attributes. The UI updates improve spacing under constrained viewports and clarify the active location state.
- **Plugin & theme developers**: No changes to public APIs, REST endpoints, or hook contracts. Experimental status limits immediate production impact unless actively prototyping dashboard widgets.
- **Action required**: None. Location modification is explicitly deferred to the settings interface; no migration or code adjustments needed.

## Technical details

The widget’s state model promotes `location` from ephemeral session data to a persistent user attribute, ensuring persistence across page loads. Layout constraints were refactored within the widget container to prevent overflow on narrower viewports. Visual feedback for the active location was enhanced, while the inline location-change control was removed to delegate editing to the settings panel. No new REST routes, PHP functions, or client-side hooks were introduced; changes operate entirely within the experimental dashboard’s block-based attribute system.

## Contribution

Initiated by @simison as a direct follow-up to #78553 under the broader Dashboard exploration (#77616). Merged as `67ca220` with straightforward review focused on UI iteration and attribute persistence. No alternative architectural approaches were debated; the changes directly address stated usability and state-management goals.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
