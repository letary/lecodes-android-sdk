# LeCodes Android SDK

Binary distribution of the LeCodes Android runtime — a self-contained fat AAR (the Kotlin host, the
UI renderer and the native engine, `arm64-v8a` + `armeabi-v7a`; the Vulkan one `arm64-v8a`) attached to the
[Releases](../../releases) of this repo. Source lives in the private LeCodes repositories; this repo
only hosts the artifacts. Current: **2.2.12** (SDK 2.x — a 2.x release runs the bundles the 2.x
`lecodes` CLI compiles; the 1.x releases stay for 1.x projects).

**Building a LeCodes app?** You normally don't wire this up by hand — `lecodes app init android`
generates a ready Gradle shell around it (see the [lecodes CLI](https://le.codes)).

## Variants

One release carries five artifacts of the same Kotlin API; a shell links the smallest one covering
the engines its bundle uses (`lecodes app sync` picks it):

| artifact | engines |
|---|---|
| `lecodes-sdk` | full: 3D (Filament on OpenGL ES) + 2D + physics (Jolt) + navigation + multiplayer |
| `lecodes-sdk-3d` | 3D rendering without the game stack |
| `lecodes-sdk-2d` | the 2D engine, no 3D |
| `lecodes-sdk-core` | UI only |
| `lecodes-sdk-vulkan` | full, with Filament on **Vulkan** (OpenGL ES where the device has no Vulkan 1.1); `arm64-v8a` only |

`lecodes-sdk-vulkan` is for 3D games: set `android.variant: "vulkan"` in `app.json` (auto-selection
never picks it). Its APK installs on 64-bit devices only, and the OpenGL ES fallback keeps a Vulkan-less
device running.

## Install (manual)

The AAR is resolved straight off the release-download URLs — no Maven registry. In
`settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        ivy {
            url = uri("https://github.com/letary/lecodes-android-sdk/releases/download")
            patternLayout { artifact("[revision]/[artifact]-[revision].[ext]") }
            metadataSources { artifact() }
            content { includeGroup("io.letary") }
        }
    }
}
```

In your app module:

```kotlin
dependencies {
    implementation("io.letary:lecodes-sdk:2.2.12@aar")   // or lecodes-sdk-3d / -2d / -core / -vulkan
    // The AAR ships no POM, so its dependencies are declared explicitly:
    implementation("androidx.core:core-ktx:1.18.0")
    implementation("androidx.appcompat:appcompat:1.7.1")
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
}
```

Requirements: `minSdk 23`, `compileSdk 36`, Java/Kotlin target 17 (the classes are compiled for 17).

## Quick start

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val engine = LecodesEngine(applicationContext)          // keep it in a ViewModel in real apps
        setContentView(LeCodesView(engine = engine, context = applicationContext))
        engine.onFetchLocal = { path -> assets.open(path).use { it.readBytes() } }   // the bundle's assets
        engine.attach(this)                                     // permission prompts, system back
        engine.setLaunchUrl(intent.data?.toString())            // BEFORE run() — read synchronously at boot
        engine.run(assets.open("app.js").bufferedReader().use { it.readText() })
    }
}
```

Extension points: `engine.registerCallback` (sync globals), `engine.registerService` /
`engine.registerView` (the channels LeCodes plugins use).

## Permissions

The AAR's manifest merges `ACCESS_NETWORK_STATE` and `VIBRATE` into consumers. Everything else
(INTERNET, CAMERA, location, notifications) is declared by the app or by installed plugins.

## Verifying downloads

Every release carries a `checksums-<version>.txt` asset with the AARs' SHA-256.
