# Android Security Lints

This repository contains custom lint checks for Android development. These lint
checks are by nature more security-focused and experimental than the built-in
lint checks within Android Studio, and are intended for more security-conscious
developers.

These lint checks are based on guidance from the
[Android Application Security Knowledge Base](https://developer.android.com/privacy-and-security/risks)
that the Android Vulnerability Research team has developed, and common recurring
vulnerabilities that the team spots in the wild.

Visit the official
[Android Lint Github Repo](https://github.com/googlesamples/android-custom-lint-rules)
for guidance on writing your own custom lint checks.

This library uses the Apache license, as is Google's default.

## How to use this library

This library is available
on [![Google's Maven Repository](https://img.shields.io/maven-metadata/v?metadataUrl=https%3A%2F%2Fdl.google.com%2Fandroid%2Fmaven2%2Fcom%2Fandroid%2Fsecurity%2Flint%2Flint%2Fmaven-metadata.xml&label=Google's%20Maven%20Repository)](https://maven.google.com/web/index.html#com.android.security.lint:lint)

1. Add the dependency to the app directory's `build.gradle` file:

   ```groovy
   dependencies {
     lintChecks 'com.android.security.lint:lint:<version>'
   }
   ```

   If using Kotlin instead of Groovy, add the dependency to the app directory's `build.gradle.kts` file:
   ```kotlin
   dependencies {
      lintChecks("com.android.security.lint:lint:<version>")
   }
   ```

2. Run `./gradlew lint` to see the results. Please file an issue if these instructions do not work for you.

## Lint checks included in this library

### [MASVS-STORAGE](https://mas.owasp.org/MASVS/05-MASVS-STORAGE/)

| Lint Issue ID                   | Detector                                                                                                                 | Risk                                                                                                                         |
|---------------------------------|--------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| `ExposedRootPath`               | [`MisconfiguredFileProviderDetector`](checks/src/main/java/com/example/lint/checks/MisconfiguredFileProviderDetector.kt) | Allowing the root directory of the device in the configuration provides arbitrary access to files and folders for attackers. |
| `SensitiveExternalPath`         | [`MisconfiguredFileProviderDetector`](checks/src/main/java/com/example/lint/checks/MisconfiguredFileProviderDetector.kt) | Sensitive info like PII should not be stored outside of the application container or system credential storage facilities.   |
| `LogInfoDisclosure`             | [`LogcatDetector`](checks/src/main/java/com/example/lint/checks/LogcatDetector.kt)                                       | Logging potentially sensitive information to logcat can leak it.                                                             |                                                     |
| `DotPathAttribute`              | [`MisconfiguredFileProviderDetector`](checks/src/main/java/com/example/lint/checks/MisconfiguredFileProviderDetector.kt) | Sharing broad path ranges can expose sensitive files by mistake.                                                             |                                                     |
| `SlashPathAttribute`            | [`MisconfiguredFileProviderDetector`](checks/src/main/java/com/example/lint/checks/MisconfiguredFileProviderDetector.kt) | Sharing broad path ranges can expose sensitive files by mistake.                                                             |                                                     |
| `HardcodedAbsolutePath`         | [`MisconfiguredFileProviderDetector`](checks/src/main/java/com/example/lint/checks/MisconfiguredFileProviderDetector.kt) | Sharing broad path ranges can expose sensitive files by mistake.                                                             |                                                     |
| `InsecureLegacyExternalStorage` | [`InsecureStorageDetector`](checks/src/main/java/com/example/lint/checks/InsecureStorageDetector.kt)                     | Opting out of Android Scoped Storage allows arbitrary access to the app's files in external storage.                         |                                                     |


### [MASVS-CRYPTO](https://mas.owasp.org/MASVS/06-MASVS-CRYPTO/)

| Lint Issue ID                | Detector                                                                                                       | Risk                                                                                                                                        |
|------------------------------|----------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| `VulnerableCryptoAlgorithm`  | [`BadCryptographyUsageDetector`](checks/src/main/java/com/example/lint/checks/BadCryptographyUsageDetector.kt) | Using weak or broken cryptographic hash functions may allow an attacker to reasonably determine the original input.                         |
| `UnsafeCryptoAlgorithmUsage` | [`BadCryptographyUsageDetector`](checks/src/main/java/com/example/lint/checks/BadCryptographyUsageDetector.kt) | Using insecure modes and paddings with cryptographic algorithms is unsafe and vulnerable to attacks.                                        |
| `WeakPrng`                   | [`WeakPrngDetector`](checks/src/main/java/com/example/lint/checks/WeakPrngDetector.kt)                         | Using non-cryptographically secure PRNGs in security contexts like authentication allows attackers to guess the randomly-generated numbers. |

### [MASVS-NETWORK](https://mas.owasp.org/MASVS/08-MASVS-NETWORK/)

| Lint Issue ID             | Detector                                                                                                                       | Risk                                                                                                                                                                                                                        |
|---------------------------|--------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DefaultCleartextTraffic` | [`MissingNetworkSecurityConfigDetector`](checks/src/main/java/com/example/lint/checks/MissingNetworkSecurityConfigDetector.kt) | On API level 27 and below, the default network security config trusts cleartext traffic and needs to be explicitly opted out by the application to only use secure connections.                                             |
| `DefaultTrustedUserCerts` | [`MissingNetworkSecurityConfigDetector`](checks/src/main/java/com/example/lint/checks/MissingNetworkSecurityConfigDetector.kt) | On API level 23 and below, the default network security config trusts user-added CA certificates. In practice, it is better to limit the set of trusted CAs so only trusted CAs are used for an app's secure connections.   |
| `DisabledAllSafeBrowsing` | [`SafeBrowsingDetector`](checks/src/main/java/com/example/lint/checks/SafeBrowsingDetector.kt)                                 | Safe Browsing is a service to help applications check URLs against a known list of unsafe web resources. We recommend keeping Safe Browsing enabled at all times and designing your app around any constraints this causes. |
| `InsecureDnsSdkLevel`     | [`DnsConfigDetector`](checks/src/main/java/com/example/lint/checks/DnsConfigDetector.kt)                                       | Custom DNS configurations can be insecure, making an application susceptible to DNS spoofing attacks. We recommend instead making use of Android OS's built-in transport security on SDK levels 28 and above.               |

### [MASVS-PLATFORM](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM/)

| Lint Issue ID                        | Detector                                                                                               | Risk                                                                                                                                                                                                  |
|--------------------------------------|--------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `TapjackingVulnerable`               | [`TapjackingDetector`](checks/src/main/java/com/example/lint/checks/TapjackingDetector.kt)             | Views without the `filterTouchesWhenObscured` attribute are susceptible to tapjacking attacks by other apps obscuring the UI to trick the user into performing certain actions.                       |
| `StrandhoggVulnerable`               | [`StrandhoggDetector`](checks/src/main/java/com/example/lint/checks/StrandhoggDetector.kt)             | Android previously had a bug in task reparenting in earlier versions, allowing malicious applications to hijack legitimate user actions and trick users into providing credentials to malicious apps. |
| `MissingAutoVerifyAttribute`         | [`CustomSchemeDetector`](checks/src/main/java/com/example/lint/checks/CustomSchemeDetector.kt)         | Exporting sensitive functionality via custom URL schemes should be properly protected by making use of app link verification.                                                                         |
| `InsecureStickyBroadcastsPermission` | [`StickyBroadcastsDetector`](checks/src/main/java/com/example/lint/checks/StickyBroadcastsDetector.kt) | Sticky broadcasts provide no security as they can be accessed, sent, and modified by anyone.                                                                                                          |                                                                                                                          |
| `InsecureStickyBroadcastsMethod`     | [`StickyBroadcastsDetector`](checks/src/main/java/com/example/lint/checks/StickyBroadcastsDetector.kt) | Sticky broadcasts provide no security as they can be accessed, sent, and modified by anyone.                                                                                                          |                                                                                                                          |

### [MASVS-CODE](https://mas.owasp.org/MASVS/10-MASVS-CODE/)

| Lint Issue ID                            | Detector                                                                                                       | Risk                                                                                                                                                                                                                                   |
|------------------------------------------|----------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `InsecurePermissionProtectionLevel`      | [`PermissionDetector`](checks/src/main/java/com/example/lint/checks/PermissionDetector.kt)                     | Custom permissions should have a `signature` protectionLevel or higher.                                                                                                                                                                |
| `UnintendedExposedUrl`                   | [`UnintendedExposedUrlDetector`](checks/src/main/java/com/example/lint/checks/UnintendedExposedUrlDetector.kt) | URLs that look intended for debugging and development purposes only are exposed in the application, allowing attackers to gain access to parts of the application and server that should be kept secure.                               |
| `UnintendedPrivateIpAddress`             | [`UnintendedExposedUrlDetector`](checks/src/main/java/com/example/lint/checks/UnintendedExposedUrlDetector.kt) | Private IP addresses are referenced that may have been intended only for debugging and development, and should not be exposed publicly.                                                                                                |
| ~~`UnsanitizedContentProviderFilename`~~ | ~~[`UnsafeFilenameDetector`](checks/src/main/java/com/example/lint/checks/UnsafeFilenameDetector.kt)~~         | ~~Trusting ContentProvider filenames without sanitization may lead to unintended file writes.~~ This lint check is now built into Android Studio under the issue ID `UnsanitizedFilenameFromContentProvider`. Please use this instead. |                                                                         |
| `ExtendedBluetoothDiscoveryDuration`     | [`BluetoothAdapterDetector`](checks/src/main/java/com/example/lint/checks/BluetoothAdapterDetector.kt)         | An overly long discovery time can increase the timeframe in which attackers can conduct Bluetooth attacks.                                                                                                                             |                                                                         |
| `ZeroBluetoothDiscoveryDuration`         | [`BluetoothAdapterDetector`](checks/src/main/java/com/example/lint/checks/BluetoothAdapterDetector.kt)         | A discovery time of zero causes the device to be discoverable as long as the application is running in the background or foreground.                                                                                                   |                                                                         |

## Contact

For questions, comments or feature requests, please file an issue or start a
discussion on GitHub. We would love to hear from you.
