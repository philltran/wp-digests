# React 19 Upgrade in WordPress

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Jarda Snajdr
- **Published:** 2026-05-27
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/05/27/react-19-upgrade-in-wordpress/](https://make.wordpress.org/core/2026/05/27/react-19-upgrade-in-wordpress/)

## Summary

WordPress Core is upgrading its underlying client-side UI library from React 18 to React 19, with initial rollout in the Gutenberg plugin (v23.3) and targeting full integration in WordPress 7.1. This migration removes several long-deprecated rendering APIs, shifts behavioral patterns around ref callbacks and function component props, and enforces stricter TypeScript inference, requiring developers to update client-side code to maintain compatibility and leverage new capabilities.

## Impact

- **Plugin & Theme Developers:** Must migrate away from removed `@wordpress/element` exports (`render`, `hydrate`, `unmountComponentAtNode`) and replace `findDOMNode` usages with refs. Function components no longer support `defaultProps`; use ES6 default parameters instead. Update any `inert` attribute assignments to booleans. Remove unnecessary `forwardRef()` wrappers as function components now accept `ref` directly. Ref callbacks that previously returned non-undefined values will now be interpreted as cleanup functions, potentially altering DOM lifecycle behavior.
- **TypeScript Consumers:** The `MutableRefObject` type is deprecated and inference rules for `useRef`/`RefObject` have changed. `ReactElement` props are now typed as `unknown` instead of `any`, surfacing previously silent prop access issues in patterns like `cloneElement`. Resolve naming conflicts between native HTML attributes and custom component props that may now trigger compilation errors.
- **Hosting, Platform & Headless/REST Consumers:** No direct server-side impact. Client-side rendering relies on the Gutenberg plugin version for migration. Enable `SCRIPT_DEBUG` during testing to capture improved error reporting early. Test iframe interactions carefully due to subtle behavioral changes in frame communication.

## Technical details

The upgrade targets the `@wordpress/element` package (served via the `wp-element` script), shifting WordPress's React dependency to v19. Key implementation shifts include:
- **Removed Render/Hydration APIs:** `ReactDOM.render()` and `ReactDOM.hydrate()` are removed; use `createRoot()` and `hydrateRoot()`. `ReactDOM.unmountComponentAtNode()` is replaced by `root.unmount()`.
- **Ref & Lifecycle Behavior:** Ref callbacks can now optionally return cleanup functions, executed when the DOM node unmounts. Existing ref callbacks returning non-undefined values will trigger this new pattern.
- **Attribute Type Enforcement:** The `inert` attribute changed from string to boolean type checking.
- **New Hooks & Utilities:** `use`, `useActionState`, `useOptimistic`, `useFormStatus`, `Activity` (from 19.2), and `useEffectEvent` are now available through `@wordpress/element`.
- **TypeScript Enforcement:** `MutableRefObject` is deprecated. `ReactElement` props shift to `unknown`, requiring explicit type assertion or interface compliance for cloned elements.

Code pattern migration example:
```javascript
// Before (deprecated)
import { render } from '@wordpress/element';
render(<App />, document.getElementById('root'));

// After (v19)
import { createRoot } from '@wordpress/element';
const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

## Contribution

The migration was prepared during the WordPress 6.6 release cycle, which shipped React 18.3 with deprecation warnings to give developers a full release cycle for testing. The core implementation is tracked in Gutenberg PR #61521 and issue #71336, targeting Gutenberg v23.3 and WordPress 7.1. Key contributors include @tyxla, @Mamaduka, @jsnajdr, @ellatrix, @aduth, @youknowriad, and @mciampini. The team proceeded with the full upgrade after the extended deprecation period to align with upstream React releases, opting for an iterative rollout through the Gutenberg plugin before core integration.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
