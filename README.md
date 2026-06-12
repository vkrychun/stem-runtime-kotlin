# StemRuntimeSDK

Your AI can now ship complete native Android features, not just code snippets. **StemJSON** is a declarative language describing a full feature — screens, interactions, data, navigation — and **StemRuntimeSDK** runs it as native Jetpack Compose on-device. AI authors the feature; users get native Android.

![Android](https://img.shields.io/badge/Android-7.0%2B-3DDC84?logo=android)
![Kotlin](https://img.shields.io/badge/Kotlin-2.0%2B-7F52FF?logo=kotlin)
![Compose](https://img.shields.io/badge/Jetpack%20Compose-BoM%202024.12-4285F4?logo=jetpackcompose)
![Gradle](https://img.shields.io/badge/Gradle-8.x-02303A?logo=gradle)
![License](https://img.shields.io/badge/license-Proprietary%20Freeware-lightgrey)

---

## Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Zip-Packaged Modules](#zip-packaged-modules)
- [Core API](#core-api)
- [State Observation & Events](#state-observation--events)
- [Module Lifecycle](#module-lifecycle)
- [License & Watermark](#license--watermark)
- [Navigation Embedding](#navigation-embedding)
- [Custom Repositories](#custom-repositories)
- [Custom Services](#custom-services)
- [Error Handling](#error-handling)
- [Diagnostics & Logging](#diagnostics--logging)
- [Module JSON](#module-json)
- [Coroutines & Threading](#coroutines--threading)
- [Privacy & Security](#privacy--security)
- [Contributing](#contributing)
- [License](#license)

---

## Requirements

| Dependency | Minimum |
|---|---|
| Android | 7.0 (API 24) |
| compileSdk | 35 |
| Kotlin | 2.0 |
| Android Gradle Plugin | 8.0 |
| Jetpack Compose | BoM 2024.12.01 |
| Core-library desugaring | enabled (`desugar_jdk_libs` 2.0+) |

Desugaring is required because the SDK uses `java.time.*` on `minSdk 24`. Without it, `:checkDebugAarMetadata` fails with a clear message.

---

## Installation

### Maven repository

Add the Maven mirror to `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://raw.githubusercontent.com/vkrychun/stem-runtime-kotlin/main") }
    }
}
```

### App module

In your app module's `build.gradle.kts`:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
}

android {
    compileSdk = 35
    defaultConfig { minSdk = 24 }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
        isCoreLibraryDesugaringEnabled = true
    }

    buildFeatures { compose = true }
}

dependencies {
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.1.4")
    implementation("com.stemjson:stem-runtime-sdk:1.0.2")
}
```

The AAR's POM and Module Metadata bring the transitive Compose, Coil, and Media3 graph — no extra `implementation` lines needed.

---

## Quick Start

```kotlin
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.platform.LocalContext
import com.stemjson.runtime.StemRender
import com.stemjson.runtime.StemRuntime
import com.stemjson.runtime.StemValidationOutcome

class DashboardActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent { MaterialTheme { Dashboard() } }
    }
}

@Composable
private fun Dashboard() {
    val context = LocalContext.current
    val runtime = remember { StemRuntime(context) }
    var render by remember { mutableStateOf<StemRender?>(null) }

    LaunchedEffect(Unit) {
        val bytes = context.assets.open("dashboard.json").use { it.readBytes() }
        val outcome = runtime.validate(bytes)
        if (outcome is StemValidationOutcome.Success) render = outcome.render
    }

    render?.Render() ?: CircularProgressIndicator()
}
```

Three steps in practice: create a runtime, validate JSON bytes (single file or zip-packaged module), embed the returned render — `StemRender.Render()` is a `@Composable` function. The SDK accepts either a single `.json` file or a zip-packaged module and picks the loader from the byte stream — no flag required.

---

## Zip-Packaged Modules

Use a zip when a module needs bundled assets, localisation, or sub-modules.

```
my_feature.zip
├── main.json                  ← required — the module root
├── details.json               ← sub-module, loaded via file://details.json
├── localization/
│   ├── en.strings             ← "key" = "value"; format
│   └── uk.strings
└── assets/
    └── logo.png               ← loaded via file://assets/logo.png
```

- Package resources are referenced with `file://<relative-path>` and take precedence over host-app resources with the same path.
- A zip without `main.json` at the root fails validation.
- `.strings` files under `localization/` back `l10n://` sources and the `localize(key, fallback)` expression function. The runtime falls back to the host app's resources if a key is missing.

See [StemJSON Specification §14](https://github.com/vkrychun/StemJSON/blob/main/spec/v1.0.md#14-package--distribution) for the full package format.

---

## Core API

### `StemRuntime`

The entry point. Create one per app or feature scope.

```kotlin
public class StemRuntime(
    context: Context,
    public val configuration: Diagnostics.Configuration = Diagnostics.Configuration(),
)
```

```kotlin
// Default — picks up the application context for you
val runtime = StemRuntime(context)

// With diagnostics
val runtime = StemRuntime(
    context,
    Diagnostics.Configuration(enabled = true, minLevel = StemSeverity.WARNING),
)
```

All chainable configurators return `this`, so they compose fluently:

```kotlin
val runtime = StemRuntime(context)
    .navigationEmbedded()
    .license("…")
    .watermarkPosition(StemWatermarkPosition.TopEnd)
```

For repository / service / picker registration, see the [Custom Repositories](#custom-repositories) and [Custom Services](#custom-services) sections.

### Validation

```kotlin
public suspend fun validate(
    bytes: ByteArray,
    ignore: Set<StemSeverity> = emptySet(),
): StemValidationOutcome
```

`ignore` suppresses the listed severity levels from causing a `Failure` (e.g. `setOf(StemSeverity.WARNING, StemSeverity.NOTE)`).

`StemValidationOutcome` is a sealed interface; the validation report is **present on both branches** — even a successful validation may carry advisory notes/warnings:

```kotlin
public sealed interface StemValidationOutcome {
    public val report: StemValidationReport

    public data class Success(val render: StemRender, override val report: StemValidationReport) : StemValidationOutcome
    public data class Failure(override val report: StemValidationReport) : StemValidationOutcome

    public fun renderOrNull(): StemRender?
    public fun onSuccess(block: (StemRender) -> Unit): StemValidationOutcome
    public fun onFailure(block: (StemValidationReport) -> Unit): StemValidationOutcome
}
```

Typical handling:

```kotlin
runtime.validate(bytes)
    .onSuccess { render = it }
    .onFailure { Log.w("Stem", it.render()) }
```

`StemValidationReport.render()` is the formatted, human- and machine-readable form:

```
=== Validation Report: 2 errors, 1 warning ===
[V002] [error] login_btn V002: Value 'repositoryId' is missing
[N007] [warning] product_list N007: Endpoint returned 404 for products.json
```

Each line: `[code] [severity] <path> <code>: <message>`. The format is designed for **AI-in-the-loop authoring**: feed the report back to the model and it will revise the StemJSON module until validation passes.

### `StemRender`

The renderable handle returned by `validate`. Embed it via its `@Composable Render()` function or read named context values through the indexed-access operator:

```kotlin
// 1. Embed in Compose
render.Render()

// 2. Read named context values declared in the module's JSON `context`
val title: String? = render["title"]
val icon:  String? = render["icon"]
```

`render.Render()` is the only composition entry point. It runs the SDK integrity checks and wraps the module body with the unlicensed-build watermark on every composition — both happen unconditionally and cannot be skipped by host code.

Each `StemRender` carries a stable `unitId` so it can be used as a key in `LaunchedEffect`, `DisposableEffect`, `remember(render.unitId) { … }`, and Compose recomposition.

---

## State Observation & Events

### Subscribe to a state key (callback form)

```kotlin
public fun subscribe(
    key: String,
    render: StemRender,
    handler: (Any?) -> Unit,
): Closeable
```

```kotlin
val handle: Closeable = runtime.subscribe("cartCount", render) { value ->
    updateBadge(value as? Long ?: 0L)
}
// later
handle.close()
```

### Stream state changes (Flow form)

```kotlin
public fun stream(key: String, render: StemRender): Flow<Any?>
```

```kotlin
LaunchedEffect(render.unitId) {
    runtime.stream("cartCount", render).collect { value ->
        updateBadge(value as? Long ?: 0L)
    }
}
```

Emitted values are JSON-shaped — `null`, `String`, `Long`, `Double`, `Boolean`, `List<Any?>`, or `Map<String, Any?>`. Cast to your expected type inside the collector. The stream is `distinctUntilChanged`, so identical consecutive values do not re-emit.

`runtime.subscribe` is preferable when the consumer is not already inside a coroutine scope; `runtime.stream` is preferable when composing with other flows or applying operators.

### Trigger events from native code

```kotlin
public fun trigger(event: String, data: Any)
```

```kotlin
runtime.trigger("themeChanged", mapOf("mode" to "dark"))
```

`data` accepts any JSON-shaped value — a `Map<String, Any?>` payload is typical, but a primitive or list is also fine. The value is bound into the matching `onCustom` handler's context. Inside the module JSON, read fields as `@{<action.id>.<field>}`. Always pass every field the handler needs in the payload — path predicates with `@{…}` are not supported inside filter values (see StemJSON spec §6.2.1).

---

## Module Lifecycle

A module cannot terminate itself — it only mutates its own state. The host observes a sentinel state key and calls `kill`:

```kotlin
public suspend fun kill(render: StemRender)
```

```kotlin
LaunchedEffect(render.unitId) {
    runtime.stream("onClose", render).collect { value ->
        if (value == true) {
            runtime.kill(render)
            onDismiss()
        }
    }
}
```

`kill` resets the rendered module to its initial JSON state. The render handle stays valid — composing it again shows the module fresh, with an empty navigation stack and the JSON-declared initial state. Inherited context, registered dependencies, named actions, packaged localisation, and the component palette are retained.

To fully release every runtime-held resource (cancel coroutines, drop every store), call `release()`:

```kotlin
public fun release()
```

After `release()` the runtime instance is unusable; create a fresh `StemRuntime(context)` for a new feature scope.

---

## License & Watermark

Unlicensed builds display a small "Powered by StemJSON" badge over every rendered module on physical devices. A valid license key suppresses the badge; the corner is configurable:

```kotlin
val runtime = StemRuntime(context)
    .license("a1b2c3d4e5f6…")                     // suppresses the badge
    .watermarkPosition(StemWatermarkPosition.TopEnd) // moves it (no-op when licensed)
```

`StemWatermarkPosition` values: `BottomEnd` (default), `BottomStart`, `TopEnd`, `TopStart`. End/Start honour the device locale's layout direction.

The license key is issued for your app's `packageName`; an invalid or wrong-package key is dropped silently. Removing the badge requires a valid key — `watermarkPosition` only moves it.

---

## Navigation Embedding

By default a module creates its own navigation stack. When the module is pushed **inside a host navigation flow**, call `.navigationEmbedded()` so internal `navigation` components participate in the host's stack instead:

```kotlin
val runtime = StemRuntime(context).navigationEmbedded()
```

With this enabled, `link` destinations and `navigate push` actions land on the host's stack; system back and pop operations sync automatically.

> **`link.destination` must have `"type": "module"`.** A `scroll` / `vstack` placed there renders but its `events` (notably `onAppear`) will not fire. Always wrap pushed layouts as `{ "type": "module", "state": {…}, "children": [ … ] }`. See StemJSON spec §link.

Do **not** use `.navigationEmbedded()` for self-contained modules (tab root, modal presentation) — they manage their own navigation.

---

## Custom Repositories

Register custom factories via:

```kotlin
public fun register(kind: StemDependencyKind, factory: StemDependencyFactory): StemRuntime
```

The factory receives a `StemDependencyContext` (`id`, `category`, `kind`, JSON-shaped `config: Map<String, Any?>`) and returns a `StemDependable`.

Built-in repositories, registered automatically and overrideable:

| Key | Built-in implementation |
|---|---|
| `StemRepositoryType.REMOTE` | OkHttp HTTP/REST client |
| `StemRepositoryType.LOCAL` | SQLite-backed document storage |
| `StemRepositoryType.SECURED` | EncryptedSharedPreferences via Tink |
| `StemRepositoryType.PHOTOS` | Android Photo Picker — `StemRender.Render()` auto-registers the launcher; no host wiring needed |

The `StemRepository` contract:

```kotlin
public interface StemRepository : StemDependable {
    public suspend fun read(params: Map<String, Any?>): Any?
    public suspend fun create(params: Map<String, Any?>): Any?
    public suspend fun update(params: Map<String, Any?>): Any?
    public suspend fun delete(params: Map<String, Any?>): Any?
}
```

`params` and return values are JSON-shaped (`String` / `Long` / `Double` / `Boolean` / `null` / `List<Any?>` / `Map<String, Any?>`). Throw `StemActionException(code, message)` to route to the action's `output.failure` chain — `code` and `message` get bound under `@{actionId.code}` / `@{actionId.message}`.

Implement only the operations you support by extending `StemRepositoryAdapter` — unimplemented methods throw `unsupportedOperation` for you:

```kotlin
class ProductRepository(id: String, private val baseUrl: String) : StemRepositoryAdapter(id) {
    override suspend fun read(params: Map<String, Any?>): Any? = okHttp.get("$baseUrl/products")
}
```

Registration follows the same shape as services — either override a built-in kind, or define your own `StemDependencyKind`:

```kotlin
// Override the built-in REMOTE repository for every module that declares `kind: "remote"`
runtime.register(StemRepositoryType.REMOTE) { ctx ->
    ProductRepository(ctx.id, ctx.config["baseUrl"] as String)
}

// Or define a new repository kind:
enum class AppRepository(override val kind: String) : StemDependencyKind {
    PRODUCTS("products");

    override val category: String = "repository"
}

runtime.register(AppRepository.PRODUCTS) { ctx ->
    ProductRepository(ctx.id, ctx.config["baseUrl"] as String)
}
```

The matching JSON `dependencies` entry references it by `category` + `kind`:

```json
{
  "dependencies": [
    {
      "id": "productsApi",
      "category": "repository",
      "kind": "products",
      "config": { "baseUrl": "https://example.com/v1" }
    }
  ]
}
```

For streaming sources (WebSocket, Firestore listener, SSE), also implement `StemListenable` to back the spec's `listen` action. `listen(params)` returns a `Flow<Any?>` of JSON-shaped values:

```kotlin
class TickerRepository(
    override val id: String,
    private val socket: WebSocket,
) : StemRepositoryAdapter(id), StemListenable {
    override fun listen(params: Map<String, Any?>): Flow<Any?> =
        socket.observe(channel = params["channel"] as String)
}
```

The dispatcher cancels the underlying coroutine when the listen action is cancelled or the module is killed, so cooperatively-cancellable `collect` is all the implementation needs.

### Custom photo picker

The `photos` repository works out of the box — `StemRender.Render()` registers an Android Photo Picker launcher for the lifetime of every composition. Override it only to swap in an in-app gallery, a pre-curated set, or a test double. Implement `StemPhotoPicker.pick(selectionLimit, mediaTypes)` to return a list of URI strings:

```kotlin
class CuratedGalleryPicker(private val library: List<String>) : StemPhotoPicker {
    override suspend fun pick(
        selectionLimit: Int,
        mediaTypes: List<String>,
    ): List<String> = library.take(selectionLimit)
}

runtime.setPhotoPicker(CuratedGalleryPicker(myUriList))
```

`setPhotoPicker` returns the runtime for fluent chaining and accepts `null` to clear a previously installed custom picker (the runtime falls back to the auto-registered default).

Worked patterns live in [`stem-examples-kotlin`](https://github.com/vkrychun/stem-examples-kotlin).

---

## Custom Services

Services handle operations outside CRUD semantics — analytics, biometrics, camera, deep links, health, and so on.

Built-in services, registered automatically:

| Key | Built-in implementation |
|---|---|
| `StemServiceType.AUDIO` | System sounds and haptics (uses `VIBRATE`) |
| `StemServiceType.PUSH` | Local notifications (uses `POST_NOTIFICATIONS` on API 33+) |
| `StemServiceType.LOCATION` | `FusedLocationProviderClient` — requires `ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION` host-side |

The `StemService` contract:

```kotlin
public interface StemService : StemDependable {
    public suspend fun execute(input: Any?): Any?
}
```

`input` is the resolved JSON-shaped payload from the action — typically a `Map<String, Any?>`, sometimes a primitive. Throw `StemActionException` to trigger `output.failure`.

```kotlin
class AnalyticsService(override val id: String) : StemService {
    override suspend fun execute(input: Any?): Any? {
        analytics.track(input as Map<String, Any?>)
        return null  // fire-and-forget — return a Map / List / primitive for output.success
    }
}
```

Define a new dependency kind for services the SDK doesn't ship as a built-in, then register your factory against it:

```kotlin
enum class AppService(override val kind: String) : StemDependencyKind {
    ANALYTICS("analytics"),
    BIOMETRICS("biometrics");

    override val category: String = "service"
}

runtime.register(AppService.ANALYTICS) { ctx ->
    AnalyticsService(ctx.id)
}
```

The matching JSON `dependencies` entry uses the `category` + `kind` discriminator:

```json
{
  "dependencies": [
    { "id": "appAnalytics", "category": "service", "kind": "analytics" }
  ]
}
```

To replace a built-in (a host with its own audio engine, a test double for push, etc.), pass the built-in `StemServiceType` to `register` instead of your custom enum — the rest of the call is identical.

Worked patterns live in [`stem-examples-kotlin`](https://github.com/vkrychun/stem-examples-kotlin).

---

## Error Handling

Validation surfaces a `StemValidationReport`; runtime failures inside repositories and services surface as `StemActionException`.

```kotlin
when (val outcome = runtime.validate(bytes)) {
    is StemValidationOutcome.Success -> render = outcome.render
    is StemValidationOutcome.Failure -> Log.e("Stem", outcome.report.render())
}
```

`StemActionException`:

```kotlin
public class StemActionException(public val code: String, message: String) : RuntimeException(message)
```

Throw it from any `StemRepository` / `StemService` / `StemListenable` implementation to route the action chain to its `output.failure` handler. The action binds `@{actionId.code}` and `@{actionId.message}` from the exception. Code values follow the catalogue in StemJSON spec §15 — e.g. `notAuthenticated`, `notFound`, `noConnection`, `keychainAccess`. Any other thrown exception is treated as an unknown failure with `code = "unknown"` and the exception's message.

`StemIssueCode` (returned in `StemIssue` entries inside the report) is the validator-side catalogue: `V###` for structural issues, `E###` for general, `N###` for network, `S###` for storage, `K###` for security. The code string is stable across releases.

---

## Diagnostics & Logging

```kotlin
public object Diagnostics {
    public data class Configuration(
        val enabled: Boolean = true,
        val minLevel: StemSeverity = StemSeverity.NOTE,
    )
}
```

```kotlin
// Default — enabled, minLevel = NOTE
val runtime = StemRuntime(context)

// Quieter — drop notes and below
val runtime = StemRuntime(
    context,
    Diagnostics.Configuration(minLevel = StemSeverity.WARNING),
)

// Silence
val runtime = StemRuntime(
    context,
    Diagnostics.Configuration(enabled = false),
)
```

Severity ladder (`StemSeverity`): `BINGO`, `INFO`, `NOTE`, `WARNING`, `ERROR`, `CRITICAL`. `ERROR` and `CRITICAL` block rendering.

Messages are emitted through `android.util.Log` under the tag `com.stem.runtime.sdk`. Filter with:

```bash
adb logcat -s com.stem.runtime.sdk
```

The runtime also routes every validation finding through the logger, so a host that never inspects the `StemValidationReport` still sees the validator's work in logcat.

---

## Module JSON

StemJSON modules are a declarative tree: every component has a `type`, optional `context`, optional `state`, and optional `children`. Values anywhere in the tree may be static, state-bound (`${field}`), context-bound (`@{key}`), or expression-evaluated (`{{ expr }}`).

```json
{ "id": "email_field", "type": "textfield",
  "context": { "_label": "Email", "_text": "${email}" } }
```

For the full component catalogue, value syntax, style options, and action types see the [**StemJSON v1.0 Specification**](https://github.com/vkrychun/StemJSON/blob/main/spec/v1.0.md).

---

## Coroutines & Threading

The runtime's internal scope uses `Dispatchers.Main.immediate` for its supervisor job. State emissions, the custom-event bus, and action dispatch hop through that scope.

`suspend` entry points:
- `runtime.validate(bytes, ignore)`
- `runtime.kill(render)`
- `StemRepository.read` / `.create` / `.update` / `.delete`
- `StemService.execute(input)`
- `StemListenable.listen(params)` returns a `Flow<Any?>`

Repository and service implementations are invoked from the main dispatcher — long-running work should be moved off-main internally with `withContext(Dispatchers.IO) { … }` so the UI stays responsive.

`runtime.stream(key, render)` returns a cold `Flow<Any?>` whose values are JSON-shaped Kotlin primitives, `List<Any?>`, or `Map<String, Any?>`. Cast to your expected type inside the collector. The flow is `distinctUntilChanged`, so identical consecutive values do not re-emit.

---

## Privacy & Security

StemRuntimeSDK runs entirely on-device. It contains no telemetry, no analytics, and no phone-home behaviour. The SDK transmits no data to Licensor.

The SDK's `AndroidManifest.xml` declares only two permissions, both for documented runtime features:

| Permission | Why |
|---|---|
| `android.permission.VIBRATE` | `audio` service haptic playback |
| `android.permission.POST_NOTIFICATIONS` | `push` service local notifications (API 33+) |

For your Application, remember to:

1. Add any extra permissions your StemJSON modules cause — for example `ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION` if you use the built-in `location` service.
2. Drive the runtime-permission flow for sensitive permissions (`POST_NOTIFICATIONS`, location, photos). The SDK reads `ContextCompat.checkSelfPermission` at action-fire time, so any grant takes effect on the next action invocation.
3. Disclose data flows your StemJSON modules cause (Keychain-equivalent secured storage, network requests, etc.) in your Play Store data-safety form.

To report a security vulnerability, see [`SECURITY.md`](.github/SECURITY.md).

---

## Contributing

This repository ships StemRuntimeSDK as a pre-compiled AAR. SDK source code is proprietary and is not published here. Bug reports, documentation fixes, and security disclosures are welcome — see [`CONTRIBUTING.md`](.github/CONTRIBUTING.md) and [`SECURITY.md`](.github/SECURITY.md).

The StemJSON data format was originated and authored by **Vasyl Krychun** and is published separately under the Open Web Foundation Agreement 1.0 at [`github.com/vkrychun/StemJSON`](https://github.com/vkrychun/StemJSON).

---

## License

Distributed under a Proprietary Freeware License. Unlicensed builds display a small "Powered by StemJSON" badge on physical devices.

See [`LICENSE`](LICENSE) for the EULA and [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md) for the attribution of the incorporated open-source components. The StemJSON format itself — originated and authored by Vasyl Krychun — is governed by the OWFa 1.0; see the [StemJSON spec repo](https://github.com/vkrychun/StemJSON).

Pricing: [stemjson.com/stemruntime#pricing](https://stemjson.com/stemruntime#pricing). Commercial enquiries: [vkrychun@stemjson.com](mailto:vkrychun@stemjson.com).
