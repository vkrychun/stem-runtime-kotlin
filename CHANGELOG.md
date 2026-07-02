# Changelog

All notable changes to **StemRuntimeSDK** are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.1.0] — 2026-07-02

### Added
- `ai` action — calls an AI provider and binds the result into a module. It POSTs a literal provider request `body` through a host-registered `remote` repository and optionally unwraps the response (`responsePath` plus a default `parseJson` JSON-parse), so the chained `@{id}` is the clean answer. Provider-agnostic — the model, prompt, and response schema live in `body`, and the API key is injected by the repository's auth interceptor (never in JSON); the same module targets OpenAI or Anthropic by pointing `provider` at a different repository. Matches [StemJSON v1.1.0](https://github.com/vkrychun/StemJSON/blob/main/spec/v1.1.md) §10.11.
- `StemRuntime.audit(...)` — opt-in static security review of a module before you run it. Without instantiating the module (no timers, network, or listeners start), it reports the capabilities the module declares — network endpoints, device services, on-device storage, repeating timers, and live data subscriptions — as severity-rated findings plus a capability summary, so a host can decide whether to load content from an untrusted author. Tunable via `StemSecurityPolicy`.
- `switch(test1, value1, …, default)` expression function — a flat multi-way conditional matching [StemJSON v1.1.0](https://github.com/vkrychun/StemJSON/blob/main/spec/v1.1.md) §8.6. Lazy/short-circuit like the ternary (only the matched value is evaluated); odd arity with a mandatory default. A module using `switch()` should declare `"version": "1.1"`.
- `random()` / `random(min, max)` and `range(n)` / `range(start, end)` expression functions for declarative randomization, matching [StemJSON v1.1.0](https://github.com/vkrychun/StemJSON/blob/main/spec/v1.1.md) §8.6.1. `map(range(n), random(a, b))` generates structured data (grids/boards) without hardcoding. `random()` is nondeterministic — resolve it once in an action/lifecycle value (frozen into state), never in a render binding.
- `validate(bytes, namespace = …)` — an optional per-module storage namespace. When supplied, a module's on-device data (its local database and secured items) is isolated to that namespace, so two modules that declare the same storage ids - or two installs of the same tool - keep separate data. Omit it for the previous shared behavior.
- Collection functions `setAt`, `removeAt`, `insertAt`, `keys`, `values`, and `removeKey` — matching [StemJSON v1.1.0](https://github.com/vkrychun/StemJSON/blob/main/spec/v1.1.md) §8.6. Edit an array element by position (`setAt` / `removeAt` / `insertAt`; negative indices count from the tail) and inspect or prune dictionaries (`keys` sorted, `values` key-aligned, `removeKey`); all return a new value. Positional editing makes piece-moving board games and index-addressed grids work directly.

### Clarified
- Chained ternaries (`a ? b : c ? d : e`) require no parentheses — already supported, now covered by tests.

### Fixed
- Typed numbers are now parsed locale-aware, so decimal input no longer reads as 0.

---

## [1.0.2] — 2026-06-12

### Changed
- Component `type` is now resolved case-insensitively, matching [StemJSON v1.0.2](https://github.com/vkrychun/StemJSON/blob/main/spec/v1.0.md) — the canonical form is lowercase. Existing modules are unaffected.

---

## [1.0.1] — 2026-06-01

### Added
- Clearer validation warnings for common authoring mistakes: unknown function names, malformed pipes, invalid `cast` sources, operators or function calls inside `${…}`/`@{…}` paths, and writes to undeclared state keys.
- `cast(int|double, 'date')` interprets numeric values as Unix-epoch seconds.
- Multiple modals per `style.modal` block (e.g. an alert and a sheet on one component).

### Fixed
- Negative path indices (`[-1]`) resolve from the end of the array.
- `map` honours the wrapped `{ region: { center, span } }` position shape (fixes off-screen annotation pin).
- `photos.read` always returns an array.

---

## [1.0.0] — 2026-05-08

Initial release. Implements the [StemJSON v1.0 specification](https://github.com/vkrychun/StemJSON/blob/main/spec/v1.0.md).

---

[1.1.0]: https://github.com/vkrychun/stem-runtime-kotlin/releases/tag/v1.1.0
[1.0.2]: https://github.com/vkrychun/stem-runtime-kotlin/releases/tag/v1.0.2
[1.0.1]: https://github.com/vkrychun/stem-runtime-kotlin/releases/tag/v1.0.1
[1.0.0]: https://github.com/vkrychun/stem-runtime-kotlin/releases/tag/v1.0.0
