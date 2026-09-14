# Samsara Network BLE SDK

Scan for and connect to Samsara hardware over Bluetooth Low Energy, and upload
the resulting telemetry.

This repository is the **published** SDK. Development happens in a private
monorepo; each release is published here.

| Platform | Status | Artifact |
| --- | --- | --- |
| Android | Available | `com.samsara.ble:samsara-ble-sdk`, served from this repo's Maven repository |
| iOS | Not yet published | A prebuilt `SamsaraBLE.xcframework` will be attached to a future release |

## Android

### Requirements

- `minSdk` 23
- Kotlin 2.1+ — the AAR ships Kotlin 2.2 metadata, and Kotlin reads metadata
  only one minor version forward

### Add the dependency

The artifacts are served as a static Maven repository from this repository's
GitHub Pages site. Add it alongside `google()` and `mavenCentral()` — the SDK's
own transitive dependencies (AndroidX, Play Services, Room) resolve from those.

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        maven { url = uri("https://samsarahq.github.io/samsara-network-ble/maven") }
        google()
        mavenCentral()
    }
}
```

```kotlin
// build.gradle.kts
dependencies {
    implementation("com.samsara.ble:samsara-ble-sdk:0.1.0")
}
```

### Permissions

The SDK declares **no permissions of its own**, so nothing is forced into your
merged manifest or your store listing. You declare the permissions for the
capabilities you want, and the SDK reports what is missing.

Scanning needs three: `BLUETOOTH_SCAN`, `BLUETOOTH_CONNECT`, and fine location.
Coarse location alone is not enough — `startScanning()` refuses without fine.
`BLUETOOTH_CONNECT` is required even though the SDK never connects to a device;
it shares the "Nearby devices" group with `BLUETOOTH_SCAN`, so it adds no extra
prompt.

**The SDK reports permission and capability state. Your application owns the
permission prompts, the permission copy, and the lifecycle UI.**

### Quick start

```kotlin
// No attach call: the SDK attached itself before Application.onCreate ran.
// Ask what's missing whenever you like — this needs no initialize().
for (group in SamsaraBleSDK.capabilities.getMissingPermissionGroups()) { /* request */ }

// Initialize at app launch.
SamsaraBleSDK.initialize(SamsaraConfig.Builder().apiKey("…").build())

val result = SamsaraBleSDK.startScanning()
if (!result.didStart) reportBlocked(result.failureReason, result.missingPermissions)
```

The public surface is `initialize()`, `startScanning()`, `stopScanning()`,
`initializationState`, `getConfig()`, `getDiagnostics()`, and `capabilities`.

`getDiagnostics()` returns a typed `SamsaraDiagnostics` — SDK version, init
state, scanning flag, and a capabilities snapshot. Call `toMap()` for flat
strings in a log line or crash report:

```kotlin
Log.i("samsara", SamsaraBleSDK.getDiagnostics().toMap().toString())
```

### Scanning is foreground-only

The SDK does not scan in the background and starts no services. Scanning stops
when your app leaves the foreground.

## License

MIT — see [LICENSE](LICENSE).

## Third-party notices

See [THIRD_PARTY_NOTICES-android.txt](THIRD_PARTY_NOTICES-android.txt).

The Android library statically compiles in Nordic Semiconductor's
Kotlin-BLE-Library and Kotlin-Util-Library (BSD-3-Clause) and Samsara's own
statechart runtime (MIT). That code reaches the application you ship, so the
notices must travel with it. The AAR carries them as an asset at
`assets/samsara-ble-sdk/THIRD_PARTY_NOTICES.txt`, which the Android Gradle
Plugin merges into your APK — surfacing them in an "Open source licenses"
screen satisfies the obligation.

## Support

sdks@samsara.com
