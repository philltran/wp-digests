# WordPress 7.0 Field Guide

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Amy Kamala
- **Published:** 2026-05-14
- **Tags:** `Core`, `General`, `7.0`, `field guide`
- **Link:** [https://make.wordpress.org/core/2026/05/14/wordpress-7-0-field-guide/](https://make.wordpress.org/core/2026/05/14/wordpress-7-0-field-guide/)

## Summary

WordPress 7.0 introduces the WP AI Client and Connectors API, establishing a provider-agnostic architecture for plugin-driven AI integration. The update ships with a unified Client-Side Abilities package, expands server-side registry management via the Connectors screen, and defaults `contentOnly` mode more broadly while introducing PHP-only block registration to streamline server-side rendering and state management. This foundational shift enables agentic workflows and centralized model routing without hardcoding provider credentials in themes or plugins.

## Impact

- **Plugin & Theme Developers**: Must migrate AI integrations to the `WP_AI_Client_Prompt_Builder` class and manage model preferences via `using_model_preference()`. Connectors must be registered through the Connectors API (`wp_connectors_init` action) rather than custom auth flows.
- **Block Authors**: Update `block.json` to set `"role": "content"` for attributes requiring editing under stricter `contentOnly` mode defaults. Register server-side-only blocks using `'autoRegister' => true` alongside a `render_callback()` to expose them to the client. Opt into pattern overrides via the `block_bindings_supported_attributes` filters.
- **Hosting & Platform Teams**: PHP 7.4+ is now enforced, and external libraries (backbone.js v1.6.1, Requests v2.0.17) are updated. No migration path required for existing plugins unless they rely on deprecated auth methods or unregistered AI connectors.

## Technical details

The AI Client centralizes request routing through the Settings > Connectors screen, storing provider metadata in a PHP registry while exposing client-side capabilities via `@wordpress/core-abilities` and `@wordpress/abilities`. The `wp_connectors_init` action allows overriding connector metadata to support new auth methods (`api_key` or `none`). In block development, `contentOnly` mode now defaults for patterns relying on unrestricted editing; opt-out is handled by the `disableContentOnlyForUnsyncedPatterns` setting or the `block_editor_settings_all` PHP filter. Block Bindings pattern overrides apply to custom blocks via the `block_bindings_supported_attributes` filters. Server-side block registration enables automatic client exposure when `'supports' => array( 'autoRegister' => true )` is declared alongside a `render_callback()`. Iframed editor stability is enforced when all inserted blocks use Block API v3 or higher.

## Contribution

Shipped during the WordPress 7.0 beta/rc phase following core review by @westonruter, @sabernhardt, @marybaum, and others. The architecture evolved from earlier agentic proposals into a finalized Connectors API backbone with provider-agnostic routing. Documentation was refined through multiple field guide edits, standardizing `contentOnly` mode defaults, PHP-only block registration syntax, and library version bumps (backbone.js, Requests, CodeMirror v5) prior to the final release candidate publication.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
