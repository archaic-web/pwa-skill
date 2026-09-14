---
name: archaic-pwa
description: Design, build, review, or refactor installable web applications using TypeScript, Deno, and browser standards directly. Use for Archaic PWA architecture and implementation; do not use for projects that have chosen a frontend framework or require broad legacy-browser compatibility.
---

# Archaic PWA

Build applications whose behavior remains understandable in terms of TypeScript, Deno, and the browser platform.

## Skill source and maintenance

The canonical source of this skill is [archaic-web/pwa-skill](https://github.com/archaic-web/pwa-skill), with `SKILL.md` at the repository root. When asked to change this skill, start from the latest repository revision, make and validate the change there, then refresh installed copies from the resulting committed revision. Treat installed copies and archived packages as distribution snapshots, not independent sources. Updating an installed copy does not imply automatic background synchronization.

## Non-negotiable stance

- Use TypeScript for application source. Prefer strict types, discriminated unions, and explicit parsing at untyped boundaries.
- Treat the browser as the application runtime and Deno as the development, tooling, test, server, and release platform.
- Use HTML, CSS, native ES modules, and Web APIs directly.
- Do not introduce a frontend framework. This includes libraries that take ownership of rendering, component lifecycle, routing, application state, dependency injection, or project structure.
- Do not compensate by growing a private framework. Avoid generic render engines, reactive runtimes, base-component hierarchies, global stores, routers, dependency containers, and convention-driven registries.
- When clean implementation would require framework-scale machinery, simplify or omit the feature, accept a narrower modern-browser baseline, or ask the user to reconsider the requirement.
- Libraries may provide contained capabilities that the platform does not provide well. Keep them behind narrow, explicit boundaries.

This is a deliberate product tradeoff, not a temporary starting point from which to recommend React, Vue, Svelte, Angular, or equivalents.

## Order of preference

Choose facilities in this order when they fit:

1. TypeScript and ECMAScript language facilities
2. HTML, CSS, and Web APIs
3. Deno built-ins for development-side and server-side work
4. Deno `@std`
5. a deliberately selected JSR library
6. an npm package only when it offers compelling, contained leverage

Do not force a higher-ranked option when a lower-ranked library removes substantial complexity without taking architectural control. Pin external dependencies, keep the lockfile, and do not load mutable third-party code from a CDN at runtime.

For each proposed runtime dependency, be ready to state:

- the capability it supplies;
- why the platform is insufficient;
- the narrow module boundary that contains it;
- whether it works in the browser, Deno, or both;
- how it affects delivery, offline behavior, and updates.

## Compatibility stance

Target an explicit modern-browser baseline derived from the actual users and deployment. Prefer feature detection and a clear unsupported state over polyfills, transpilation gymnastics, or parallel implementations. Do not promise a capability merely because it exists in one browser.

Progressive enhancement is useful where native HTML already expresses the interaction, but a JavaScript-dependent application is acceptable. Preserve normal web semantics: links navigate, buttons act, forms submit data, labels identify controls, URLs identify navigable states, and browser history works.

## Architecture

Keep architecture smaller than the application:

- Organize by application capability rather than by technical file type when that makes ownership clearer.
- Keep domain rules and use cases independent of DOM nodes, IndexedDB requests, network responses, and Deno APIs.
- Compose concrete dependencies visibly in an entry module. Prefer arguments and factory functions over lookup or injection machinery.
- Give each piece of state a single owner and an explicit lifetime. Do not create a global store by default.
- Model valid states and outcomes as types instead of coordinating booleans and nullable fields.
- Parse and validate network, storage, message, and user-input data before treating it as a domain type.
- Use direct, localized DOM updates. Use native custom elements only when their browser-defined lifecycle helps a genuinely reusable element.
- Every mounted screen or element must visibly own and release its listeners, timers, observers, subscriptions, and abortable work.

Read [architecture.md](references/architecture.md) when designing modules, navigation, state, UI composition, or deciding whether code is becoming a private framework.

## PWA behavior

Treat offline data, service workers, installation, and updates as application behavior rather than packaging details.

- Separate user data from cached HTTP resources.
- Do not describe browser storage as permanent safekeeping; define synchronization, export, or recovery for important data.
- Keep service-worker handlers restart-safe. Never rely on worker-global memory as durable state.
- Make offline behavior explicit per route and operation.
- Distinguish local commit, queued synchronization, server acceptance, and conflict rather than hiding them behind `save()`.
- Make retries idempotent where duplication would be harmful.
- Coordinate updates across page code, service-worker code, cached resources, persisted schemas, and server APIs.
- Apply updates at understandable boundaries and preserve in-progress work.

Read [pwa-lifecycle.md](references/pwa-lifecycle.md) whenever the task touches installation, offline use, persistence, caching, synchronization, service workers, or upgrades.

## Deno toolchain and delivery

Prefer one visible `deno.json` and a small canonical task surface such as `check`, `test`, `serve`, and `build`. Use Deno's formatter, linter, type checker, test runner, task runner, and HTTP facilities before assembling overlapping tools.

Compilation must be mechanical. Application semantics must not depend on compiler plugins, generated framework modules, decorator transforms, or magic file naming. Prefer native browser ESM and structure-preserving TypeScript transpilation. Introduce bundling, minification, or code splitting only for a measured delivery need or because an approved browser library cannot otherwise be delivered cleanly.

Ensure that every emitted browser import is browser-resolvable. Deno, JSR, npm, and browser module resolution are not interchangeable merely because development succeeds under Deno.

Read [toolchain-and-testing.md](references/toolchain-and-testing.md) when creating project tasks, choosing dependencies, compiling browser code, testing, or preparing a release.

## Working method

When creating or changing an application:

1. Establish the supported browsers, install/offline expectations, data authority, and whether a server exists.
2. Identify the smallest platform-native design and the domain states that deserve types.
3. Call out any requested behavior that would require a framework, a private framework, or unsupported browser capability. Offer a smaller behavior before adding machinery.
4. Implement vertical capabilities through explicit modules and direct platform APIs.
5. Verify domain behavior in fast tests and platform behavior in a real browser.
6. Test failure and recovery: direct navigation, reload, Back/Forward, offline restart, failed storage, duplicate retry, interrupted update, and old persisted data as relevant.

Do not introduce abstractions solely to make a diagram symmetrical, imitate enterprise layering, or prepare for hypothetical providers. Add a boundary when it protects a real distinction, isolates a chosen library or platform API, or enables a meaningful test.

## Definition of done

The result should have:

- a documented modern-browser baseline and intentional unsupported behavior;
- strict TypeScript with validation at external boundaries;
- no framework and no framework-shaped application layer;
- visible composition, state ownership, navigation, cleanup, and failure behavior;
- a small Deno task surface with reproducible dependencies;
- a browser-resolvable release artifact whose structure remains recognizable;
- real-browser checks for the PWA behaviors the product claims to support.
