# Application architecture

Read this reference when designing modules, navigation, state, UI composition, or reviewing whether application code is turning into a private framework.

## Shape capabilities vertically

A capability may contain its domain types, use cases, platform adapter, and UI code together. Split them only where the boundary is useful; do not manufacture one interface and one implementation for every function.

One reasonable shape is:

```text
src/
  main.ts
  app-shell.ts
  learning/
    model.ts
    use-cases.ts
    screen.ts
  progress/
    model.ts
    store.ts
    indexed-db.ts
  platform/
    navigation.ts
    service-worker-client.ts
```

This is an example, not a mandatory tree. Public entry modules are conventions rather than JPMS-style enforced boundaries. Keep imports honest and add an import-rule checker only after boundary violations become a demonstrated problem.

## Compose explicitly

The entry module should create concrete adapters, pass them to use cases or screen factories, register routes, and start the application. Ordinary function parameters and small structural interfaces are sufficient in most cases.

Avoid service locators, decorators, annotation scanning, dependency containers, and module-global dependency mutation.

## Use types to eliminate impossible states

Prefer a discriminated union:

```ts
type LessonState =
  | { kind: "loading" }
  | { kind: "active"; exercise: Exercise }
  | { kind: "completed"; score: Score }
  | { kind: "unavailable-offline" };
```

over several flags and nullable values whose combinations must be remembered. Keep state transition functions free of DOM concerns where this makes behavior easier to test.

TypeScript types do not validate runtime data. Accept `unknown` at JSON, storage, postMessage, URL, and form boundaries; check it before constructing trusted domain values. A small schema-validation library is acceptable when hand-written validation would become repetitive and error-prone, provided it remains at the boundary.

## Render directly

Prefer semantic HTML already present in the document or cloned from `<template>` elements. Create and update DOM nodes directly within the module that owns the screen. Event delegation is useful when it simplifies a dynamic collection, but is not a global convention.

Avoid making `innerHTML` the general rendering mechanism. If markup strings are genuinely useful, keep them local and ensure untrusted values are not interpreted as HTML.

Custom elements are appropriate for reusable elements whose own lifecycle and public attributes/events form a coherent browser-native boundary. Do not turn every screen or fragment into a custom element merely to obtain a component vocabulary. Avoid a mandatory base element class.

## Navigation is application state

A route owns a meaningful URL, document title, visible view, focus target, and scroll policy. It must define behavior for direct load, reload, Back, Forward, and unavailable offline data.

Use anchors for navigation. Intercept only same-origin application routes that the SPA can actually handle; preserve modified clicks, downloads, external targets, and normal link behavior. Keep the History API adapter small and explicit. Do not invent a general router if a route table and `URLPattern` or URL parsing are sufficient for the application's supported browsers.

## State ownership

Classify state by lifetime:

- DOM-local interaction state, such as focus or an open disclosure;
- screen-session state, discarded on navigation;
- application-session state, retained while the page lives;
- durable local state, stored explicitly;
- server-authoritative state, represented locally with synchronization status.

Store a fact once. Derive views from its owner rather than mirroring it in several modules. A single global observable store is not the default answer.

## Framework-pressure test

Stop and simplify when several of these appear:

- a generic virtual-DOM, template-diffing, or signal runtime;
- a universal component superclass or lifecycle protocol;
- a general state store with selectors, actions, middleware, and subscriptions;
- a router with guards, loaders, nested route conventions, and plugin hooks;
- dependency lookup by tokens, decorators, or registration;
- application behavior inferred from directory names or generated modules;
- helpers whose purpose is to conceal ordinary DOM or Web API behavior;
- substantial compatibility code for browsers outside the chosen baseline.

Do not answer this pressure by proposing a framework. First remove the abstraction, reduce the interaction, use a browser primitive more directly, split the product into ordinary pages, or omit the feature.
