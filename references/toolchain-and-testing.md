# Deno toolchain, delivery, and testing

Read this reference when creating project tasks, selecting dependencies, compiling browser code, testing, or preparing a release.

## Keep one canonical interface

Expose a small task vocabulary in `deno.json`, normally including:

- `check`: formatting verification, linting, and type checking;
- `test`: fast TypeScript tests, with browser tests separated when useful;
- `serve`: a development server with SPA fallback behavior where applicable;
- `build`: a clean, reproducible browser artifact;
- optionally `verify`: the release-level checks performed in CI.

Use Deno defaults unless a project requirement justifies configuration. Do not add ESLint, Prettier, a separate TypeScript compiler installation, or another task runner merely to reproduce Deno facilities.

Keep Deno-only files separate from browser-entry graphs. Shared modules must use APIs and dependency specifiers supported by both environments, or must pass through an explicit release step.

## Compile mechanically

For a small application, prefer type checking followed by structure-preserving transpilation to browser ES modules. Current Deno provides `deno check` and `deno transpile`; `deno transpile` emits without type checking, so a successful build must not substitute for `deno check`.

Preserve meaningful filenames and module boundaries in the artifact. Copy HTML, CSS, the manifest, icons, and other static assets deliberately. Generate a cache inventory only when the service worker's strategy needs one; keep the generation step visible and deterministic.

The browser must be able to resolve the emitted graph. In particular, `jsr:`, `npm:`, Deno-only import-map entries, package export rules, and filesystem conventions do not automatically become browser-resolvable imports. For a browser dependency, choose one explicit delivery approach:

- ship browser-ready ESM under the application's origin;
- map stable browser specifiers to locally served files with an import map;
- use a small release-only bundling step when the approved dependency graph requires it.

Do not use production imports from mutable third-party URLs. Do not bundle by reflex; measure request count, parse/compile time, caching behavior, and actual target networks first. If bundling is required, keep it a delivery transform rather than an application architecture.

## Choose libraries deliberately

A suitable library provides a bounded capability such as music notation, robust schema validation, a file-format codec, or browser automation. It should expose normal values and functions and remain replaceable at one boundary.

Reject a dependency when ordinary application work must be expressed through its component model, reactive graph, route conventions, global store, dependency container, code generator, or build plugin. Marketing labels such as “library” versus “framework” do not decide this; architectural ownership does.

Prefer `@std` for Deno-side needs it covers. Browser-side compatibility and delivery remain separate questions. An npm package is acceptable for a contained need after checking its browser format, transitive graph, maintenance, license, security posture, and effect on offline delivery.

## Testing layers

Use the smallest layer that can establish the behavior:

- pure Deno tests for domain rules, parsers, state transitions, and use cases;
- contract tests reused by genuinely replaceable adapters;
- adapter tests against real IndexedDB, Cache Storage, workers, or server endpoints when platform semantics matter;
- browser automation for navigation, DOM behavior, accessibility, installation/offline claims, worker updates, and restart recovery.

An in-memory adapter does not prove that IndexedDB reopening or migration works. A DOM emulator does not prove browser history, focus, service workers, installation, storage eviction handling, or worker lifecycles.

Choose one browser automation tool as a test dependency only when needed. Keep application code independent of it. Prefer tests of observable behavior over snapshots of large DOM trees or tests coupled to internal function names.

## Release checks

Verify the served release artifact, not only development source. Depending on the application's promises, exercise:

- deep-link load and reload;
- Back and Forward navigation;
- keyboard operation, focus movement, names, roles, and live feedback;
- first load, subsequent load, and offline restart;
- failed or denied storage;
- duplicate and reordered synchronization attempts;
- update while an older page is open;
- migration from supported old data;
- absence of network requests to undeclared third parties.

Use browser developer tools and automated audits as evidence, not as substitutes for product-specific tests.

## Authoritative references

- [Deno TypeScript](https://docs.deno.com/runtime/fundamentals/typescript/)
- [Deno configuration](https://docs.deno.com/runtime/fundamentals/configuration/)
- [Deno standard library](https://docs.deno.com/runtime/reference/std/)
- [Deno transpile](https://docs.deno.com/runtime/reference/cli/transpile/)
- [MDN JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [MDN Import maps](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script/type/importmap)
