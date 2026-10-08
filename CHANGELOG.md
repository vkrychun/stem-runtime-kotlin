# Changelog

All notable changes to **StemRuntimeSDK** are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.2.0] — 2026-10-08

Implements [StemJSON v1.2](https://github.com/vkrychun/StemJSON/blob/main/spec/v1.2.md). `StemRuntime.supportedSpec` is now `"1.2"`.

### Added
- `chart` component — bar, line, area, point and pie series drawn natively on Compose Canvas, with `style.chart`, a legend, an `animated` reveal and an `_isLoading` skeleton.
- `web` component — a native `WebView` for `https://`, `http://` and package `file://` sources, with `_allowsNavigation`, `_scrollEnabled` and `style.web.javaScript`. A `data:`, `javascript:` or unsupported source renders an empty frame. Inside a scrolling screen the page scrolls first; once it is at its edge, the next drag scrolls the screen.
- `onChange` accepts an array of observers, each firing only its own chain when its key changes.
- `navigate` operation `open` hands an `https://`, `http://`, `mailto:` or `tel:` URL to the system. An unopenable URL runs `output.failure`.
- Expression functions `slice`, `pow`, `sqrt` and `log`; `prefix` and `suffix` also work on arrays.
- `StemRuntime.validate(..., compatibility)` with `StemCompatibility.DEGRADE` (default) or `STRICT`. `STRICT` refuses a module that declares a later minor version with a single error, before anything is decoded or rendered.
- `StemRuntime.supportedSpec` — the spec revision this runtime implements, for hosts that negotiate versions.
- Security audit: `StemSecurityPolicy.hostNamespace` and finding S012 (local or secured stores that other modules share when the host passes no namespace), finding S011 for `web` sources, and `navigate open` reported as a network endpoint (S001) or a dynamic capability (S002). The audit now also walks `dynamic` prototypes, `link` destinations and conditional branches.

### Changed
- `progress` with no `_value` renders the indeterminate spinner and a missing `_total` means 1.0; neither raises a warning. The `_isLoading` skeleton is now a bar matching the determinate bar's footprint.
- An unknown component type renders a placeholder followed by its children, not an empty box.
- A vstack with `_lazy: true` renders lazily under a vertical `scroll` or `form` alongside other children. A vertical scroll nested in another vertical scroll lays out as a plain column and logs a warning.
- A `state` action with an `output.success` chain writes only to the dispatching module and no longer overwrites a same-named key in ancestor modules.
- Interval ids live in their own namespace: several event chains may reuse one interval id to cancel or restart it.
- `StemRender` repaints when the same module id is re-validated with different content.
- `StemSecurityPolicy` gained a `hostNamespace` field, so code compiled against 1.1.0 that used its default-argument constructor or `copy` must be recompiled (source-compatible).
- Host-visible messages and logs no longer carry internal or sensitive details, only the code, severity, JSON path and the module's own values. `_debug` logs resolved values only on a debuggable host, and a remote request failure reports just its HTTP status.

### Fixed
- A quoted subscript (`dict['key']` or `dict["key"]`) looks up that literal key, dots included; a quoted subscript on an array, or a number on a dictionary, resolves to none.
- A lazy grid whose `dynamic` items repeat a scalar value no longer crashes on first render.
- The security audit API (`StemAuditOutcome`, `StemSecurityReport`, `StemSecurityFinding`, `StemCapabilityManifest`, `StemSecurityPolicy`) is accessible from the published AAR, so hosts can read audit outcomes.
- The runtime's native library is linked with a 16 KB page size, as Google Play requires.
- A `navigate` `push` whose source cannot be loaded runs `output.failure`.
- A `listen` action whose dependency is not registered runs `output.failure` instead of doing nothing.
- An `image` shows its loading skeleton while the picture downloads; `_placeholder` appears only when the load fails and may be an `asset://` image.

### Conformance
- Module `version` is checked against the runtime: an absent version is a note, an unparsable one or a later minor a warning, a higher major a critical issue and the module is not rendered. Version findings are reported at `/version`.
- A later-minor construct degrades instead of blocking the module: an unknown event of any payload shape is ignored with a warning (V026), an unknown action kind is skipped with a warning (V028), an unknown dependency category is omitted with a warning (E007) and an action addressing it runs `output.failure`, and an unknown `navigate` operation runs `output.failure`.
- An unrecognised enumerated value stays an error at every declared version.
- New validation findings: `@{…}` that names no declared action (V016, error), a UI-gating flag set before an async action and never reset on failure (V018), and `${…}` of an undeclared state key (V019); unknown style domains and keys (V027); empty action objects (V028); chain keys inside an action `input` (V029); a bare or mixed `onChange` array (V032).
- A blank `navigate` `operation` is a missing required field (V002).
- `infinity` on a dimension other than a maximum is a warning, not an error.
- Piped calls such as `x |> slice(1)` no longer raise a false arity warning (V015).
- `chart` and `web` context is checked: missing data or source (V002), invalid series, axes or source scheme (V003).

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
