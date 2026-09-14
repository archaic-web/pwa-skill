# PWA lifecycle

Read this reference for installation, offline use, persistence, caching, synchronization, service workers, and upgrades.

## Begin with product promises

State which routes open offline, which operations work offline, what survives a restart, what requires an account or server, and what installation adds. “Offline-first” and “PWA” are not sufficiently precise requirements.

Declare an explicit browser baseline. Feature-detect optional capabilities and offer a clear unavailable state. Do not emulate missing platform features with framework-scale compatibility code.

## Manifest and installation

Use the Web App Manifest to describe identity, name, icons, start URL, scope, colors, and display mode. Treat platform-specific installation prompts and richer integration as optional enhancements unless the target browsers guarantee them.

An installed PWA still needs sensible behavior when opened as an ordinary browser tab. Do not make install state an architectural fork.

## Separate data categories

- Use Cache Storage for HTTP request/response resources and an explicit delivery strategy.
- Use IndexedDB for structured local application and user data.
- Keep temporary interface state in memory or the DOM.
- Treat server-authoritative data and locally authored changes as separate states when synchronization matters.

Version persisted schemas independently from the application release. Make migrations deterministic, interruption-aware, and covered by tests using old data. Do not erase incompatible user data silently.

Browser storage may be evicted or removed. For valuable user data, define at least one of synchronization, export/import, or another recovery mechanism. Request persistent storage only as an enhancement; it is not a durability guarantee.

## Service-worker discipline

Keep the worker small. Suitable responsibilities include resource delivery, explicit cache policy, update coordination, and narrowly supported background events.

Assume the worker can stop after every event. Persist required state before the event completes, use `waitUntil()` where appropriate, and do not rely on worker-global objects as durable queues or sessions.

Give caches versioned meanings and delete only caches owned by the application. Avoid indiscriminate cache-first handling. Choose strategy per resource class, for example immutable application assets, navigations, versioned lesson content, and API requests.

An offline fallback is an intentional application state, not a generic error page for every request.

## Synchronization semantics

Model states such as:

```ts
type CommitState =
  | { kind: "local" }
  | { kind: "queued"; operationId: string }
  | { kind: "accepted"; revision: string }
  | { kind: "conflict"; local: Change; remote: RecordSnapshot };
```

Use names appropriate to the domain. The point is to avoid claiming “saved” when only one stage succeeded.

Assign stable operation identifiers and make retries safe. Define ordering, conflict handling, authentication expiry, and what happens when one operation permanently fails. Background Sync may improve progress but must not be the only retry path; retry in the foreground when the application opens or regains connectivity.

## Update compatibility

Consider these versions separately:

- currently open page code;
- waiting or active service-worker code;
- cached application resources;
- persisted data schema;
- page/worker message protocol;
- server API and synchronization protocol.

Do not force a new worker to control an old page without proving compatibility. Tell the user when an update is ready when that is safer, and activate/reload at a boundary that preserves in-progress work.

Test at least the combinations the release process can create: old page/new worker, new page/old persisted data, offline old release, interrupted migration, and a queued old operation reaching a newer server.

## Trust boundary

Client-side types, routes, and storage boundaries organize code; they do not authorize actions. A server must validate payloads, authenticate identities, and enforce access independently. Treat cached sensitive data, tokens, logs, exported data, and notifications according to the product's threat model.

## Authoritative references

- [MDN Progressive Web Apps](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps)
- [MDN Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [MDN IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [MDN Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache)
- [MDN Storage quotas and eviction](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)
