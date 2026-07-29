# LeCodes Android SDK

Binary distribution of the LeCodes Android runtime — a self-contained fat AAR (Kotlin host +
native engine, `arm64-v8a` + `armeabi-v7a`) attached to the [Releases](../../releases) of this
repo. Source lives in the private LeCodes repositories; this repo only hosts the artifacts.

**Building a LeCodes app?** You normally don't wire this up by hand — `lecodes app init android`
generates a ready Gradle shell around it (see the [lecodes CLI](https://le.codes)).

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
    implementation("io.letary:lecodes-sdk:1.1.0@aar")
    // The AAR ships no POM, so its dependencies are declared explicitly:
    implementation("androidx.core:core-ktx:1.18.0")
    implementation("androidx.appcompat:appcompat:1.7.1")
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
}
```

Requirements: `minSdk 23`, `compileSdk 36`, Java/Kotlin target 11.

## Quick start

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val engine = LecodesEngine(applicationContext)          // keep it in a ViewModel in real apps
        setContentView(LeCodesView(engine = engine, context = applicationContext))
        engine.attach(this)                                     // permission prompts need the activity
        engine.setLaunchUrl(intent.data?.toString())            // BEFORE run() — read synchronously at boot
        engine.run(assets.open("app.js").bufferedReader().use { it.readText() })
    }
}
```

Extension points: `engine.registerCallback` (sync globals), `engine.registerService` /
`engine.registerView` (JSON call/event channels — what LeCodes plugins use).

## Permissions

The AAR's manifest merges `ACCESS_NETWORK_STATE` and `VIBRATE` into consumers. Everything else
(INTERNET, CAMERA, location, notifications) is declared by the app or by installed plugins.

## Verifying downloads

Every release carries a `checksums-<version>.txt` asset with the AAR's SHA-256.
