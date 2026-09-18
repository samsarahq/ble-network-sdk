# Samsara Network BLE SDK

Scan for and connect to Samsara hardware over Bluetooth Low Energy, and upload
the resulting telemetry.

This repository is the **published** SDK. Development happens in a private
monorepo; each release is published here.

| Platform | Status | Artifact |
| --- | --- | --- |
| Android | Available | `com.samsara.ble:samsara-ble-sdk`, served from this repo's Maven repository |
| iOS | Available | A prebuilt `SamsaraBLE.xcframework`, attached to each GitHub Release and resolved by Swift Package Manager |

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
        maven { url = uri("https://samsarahq.github.io/ble-network-sdk/maven") }
        google()
        mavenCentral()
    }
}
```

```kotlin
// build.gradle.kts
dependencies {
    implementation("com.samsara.ble:samsara-ble-sdk:0.1.2")
}
```

### Permissions

The SDK declares **no permissions of its own**, so nothing is forced into your
merged manifest or your store listing. You declare the permissions for the
capabilities you want, and the SDK reports what is missing.

Scanning needs three: `BLUETOOTH_SCAN`, `BLUETOOTH_CONNECT`, and fine location.
Coarse location alone is not enough — `enableScanning()` refuses without fine.
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

val result = SamsaraBleSDK.enableScanning()
if (!result.didEnable) reportBlocked(result.failureReason, result.missingPermissions)
```

The public surface is `initialize()`, `enableScanning()`, `disableScanning()`,
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

## iOS

### Requirements

- iOS 15+
- Xcode 15+ / Swift 5.9+

### Add the package

The SDK ships as a **prebuilt binary framework**. Nothing of ours compiles in
your build, and there are no transitive packages to resolve.

Xcode: **File → Add Package Dependencies…** and use

```
https://github.com/samsarahq/ble-network-sdk.git
```

Choose product **SamsaraBLE**.

Or in a `Package.swift`:

```swift
.package(
    url: "https://github.com/samsarahq/ble-network-sdk.git",
    .upToNextMinor(from: "0.1.2")
)
// .product(name: "SamsaraBLE", package: "ble-network-sdk")
```

While the SDK is `0.x`, pin with `.upToNextMinor`. `from:` means up-to-next-major,
which would let a breaking `0.x` release in.

### Permissions

As on Android, the SDK declares nothing itself and reports what is missing. You
add the usage descriptions your app needs (`NSBluetoothAlwaysUsageDescription`,
`NSLocationWhenInUseUsageDescription`) and own the prompts.

### Quick start

```swift
import SamsaraBLE

SamsaraBleSDK.shared.initialize(config: SamsaraConfig(apiKey: "…"))

for group in SamsaraBleSDK.shared.capabilities.getMissingPermissionGroups() { /* request */ }
let result = SamsaraBleSDK.shared.enableScanning()
```

### Scanning is foreground-only

Same as Android: no background scanning, no services.

## License

MIT — see [LICENSE](LICENSE).

## Third-party notices

Per platform, because the two ship different components:
[THIRD_PARTY_NOTICES-android.txt](THIRD_PARTY_NOTICES-android.txt) and
[THIRD_PARTY_NOTICES-ios.txt](THIRD_PARTY_NOTICES-ios.txt).

The Android library statically compiles in Nordic Semiconductor's
Kotlin-BLE-Library and Kotlin-Util-Library (BSD-3-Clause) and Samsara's own
statechart runtime (MIT). That code reaches the application you ship, so the
notices must travel with it. The AAR carries them as an asset at
`assets/samsara-ble-sdk/THIRD_PARTY_NOTICES.txt`, which the Android Gradle
Plugin merges into your APK — surfacing them in an "Open source licenses"
screen satisfies the obligation.

The iOS XCFramework statically links Nordic's IOS-BLE-Library (BSD-3-Clause) and
the same statechart runtime. Those notices travel in the framework's zip
alongside the binary; a future release will move them inside the framework with
an API to read them, matching Android.

## Support

sdks@samsara.com
