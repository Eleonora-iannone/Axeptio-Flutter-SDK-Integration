<h1>
  <img src="https://axeptio.imgix.net/2024/07/e444a7b2-ea3d-4471-a91c-6be23e0c3cbb.png" alt="Descrizione immagine" width="80" style="vertical-align: middle; margin-right: 10px;" />
  Axeptio Flutter SDK Documentation
</h1>

This repository demonstrates the integration of the **Axeptio Flutter SDK** into mobile applications, enabling seamless consent management for both [brands](https://support.axeptio.eu/hc/en-gb/articles/30431504788753-Geolocation) and [publishers](https://support.axeptio.eu/hc/en-gb/articles/23671149708945-1-Create-the-project), tailored to your specific requirements.

# 📑 Table of Contents
1. [Setup and Installation](#1-setup-and-installation)
   - [Android Setup](#android-setup)
   - [iOS Setup](#ios-setup)
2. [SDK Initialization](#2-sdk-initialization)
3. [App Tracking Transparency (ATT) Integration](#3-app-tracking-transparency-att-integration)
4. [SDK and Mobile App Responsibilities](#4-sdk-and-mobile-app-responsibilities)
5. [Retrieving and Managing Stored Consents](#5-retrieving-and-managing-stored-consents)
6. [Displaying the Consent Popup on Demand](#6-displaying-the-consent-popup-on-demand)
7. [Sharing Consents with Web Views](#7-sharing-consents-with-web-views)
8. [Clearing User Consent](#8-clearing-user-consent)
9. [Event Handling and Customization](#9-event-handling-and-customization)
***
# 1. 🚀Setup and Installation   
To integrate the Axeptio SDK into your Flutter project, run the following command in your terminal:
```bash
flutter pub add axeptio_sdk
```
This command will automatically add the `axeptio_sdk` dependency to your `pubspec.yaml` file and download the latest available version. After running this command, the SDK will be ready for use in your Flutter project.

Alternatively, you can manually add the dependency to your `pubspec.yaml` under the dependencies section:
```yaml
dependencies:
  flutter:
    sdk: flutter
  axeptio_sdk: ^latest_version
```
***
## Android Setup
#### Minimum SDK Version
To ensure compatibility with the Axeptio SDK, the **minimum SDK** version for your Flutter project must be set to **API level 26** (Android 8.0 Oreo) or higher. To verify and update this setting, open your project's `android/app/build.gradle` file and check the following:
```gradle
android {
    defaultConfig {
        minSdkVersion 26 // Ensure this is set to 26 or higher
    }
}
```
#### Add Maven Repository and Credentials
In order to download and include the Axeptio SDK, you'll need to configure the Maven repository and authentication credentials in your `android/build.gradle` file. Follow these steps:
1. Open the `android/build.gradle` file in your Flutter project.
2. In the `repositories block`, add the Axeptio Maven repository URL and the required credentials for authentication.

Here is the necessary configuration to add:
```gradle
allprojects {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://maven.pkg.github.com/axeptio/axeptio-android-sdk")
            credentials {
                username = "[GITHUB_USERNAME]"  // Replace with your GitHub username
                password = "[GITHUB_TOKEN]"    // Replace with your GitHub personal access token
            }
        }
    }
}
```
#### GitHub Authentication
To authenticate and access the Maven repository, you need to provide your **GitHub username** and **personal access token** in the `username` and `password` fields, respectively. To generate a personal access token (PAT) for GitHub, follow these steps:
1. Visit the GitHub website and navigate to **Settings > Developer settings > Personal access tokens**.
2. Click on **Generate new token** and select the necessary permissions (at least `read:packages` is required to access the repository).
3. Copy the generated token and paste it into the `password` field in your `build.gradle` file.
> **Note:** It is recommended to store sensitive information such as your GitHub credentials securely, using environment variables or a secure credential manager, rather than hardcoding them into your `build.gradle` file.

#### Sync Gradle
Once you've added the repository and credentials, sync your Gradle files by either running:
```
bash
flutter pub get
```
Or manually through Android Studio by clicking **File > Sync Project** with Gradle Files.

This will allow your project to fetch the necessary dependencies from the Axeptio Maven repository.

## iOS Setup
#### Minimum iOS Version
The Axeptio SDK supports **iOS versions 15.0 and higher**. To ensure compatibility with the SDK, make sure your project is set to support at least iOS 15.0. To check or update the minimum iOS version, open the `ios/Podfile` and ensure the following line is present:

```ruby
platform :ios, '15.0'ç
```
This ensures that your app targets devices running iOS 15 or later.

***
# 2. 🔧SDK Initialization
To initialize the Axeptio SDK on app startup, select either **brands** or **publishers** depending on your use case. The SDK is `initialized` through the AxeptioService enum and requires your `client_id`, `cookies_version`, and optionally a `consent_token`.
```dart
final axeptioSdkPlugin = AxeptioSdk();
await axeptioSdkPlugin.initialize(
    AxeptioService.brands, // Choose either brands or publishers
    [your_client_id],  // Your client ID
    [your_cookies_version],  // Version of your cookies policy
    [optional_consent_token],  // Optionally pass a consent token for existing user consent
);
await axeptioSdkPlugin.setupUI();  // Setup the UI for consent management
```
The **setupUI()** function will display the consent management UI once initialized.
***
# 3. App Tracking Transparency (ATT) Integration
The **Axeptio SDK** does not manage **App Tracking Transparency** (ATT) permissions. It is the responsibility of the host app to manage the **ATT** flow, ensuring that the user is prompted before consent management is handled by the SDK.
#### Steps for Integrating ATT with Axeptio SDK:
1. **Add Permission Description in `Info.plist`:**
In your iOS project, add the following key to `Info.plist` to explain why the app requires user tracking:
```xml
<key>NSUserTrackingUsageDescription</key>
<string>Explain why you need user tracking</string>
```
2. **Install App Tracking Transparency Plugin:**
```bash
flutter pub add app_tracking_transparency
```
3. **Request Tracking Authorization:**
Use the **AppTrackingTransparency** plugin to request permission before initializing the **Axeptio SDK's** consent UI. The flow checks the user’s status and takes appropriate actions based on their choice. 
```dart
try {
  TrackingStatus status = await AppTrackingTransparency.trackingAuthorizationStatus;

  // If the status is not determined, show the system's ATT request dialog
  if (status == TrackingStatus.notDetermined) {
    status = await AppTrackingTransparency.requestTrackingAuthorization();
  }

  // If the user denied tracking, update consent status accordingly
  if (status == TrackingStatus.denied) {
    await axeptioSdkPlugin.setUserDeniedTracking();
  } else {
    // Proceed with the consent UI setup
    await axeptioSdkPlugin.setupUI();
  }
} on PlatformException {
  // On Android, skip ATT dialog and proceed with the UI setup directly
  await axeptioSdkPlugin.setupUI();
}
```
***
# 4. 🗂SDK and Mobile App Responsibilities

The Axeptio SDK and your mobile application each have distinct responsibilities in the consent management process:

### Mobile App Responsibilities:

1. **Implementing ATT Flow**  
   Your app must handle the App Tracking Transparency (ATT) permission request and manage the display of the ATT prompt at the appropriate time relative to the Axeptio CMP.

2. **Privacy Declaration**  
   The app must declare data collection practices accurately in App Store privacy labels.

3. **Handling SDK Events**  
   The app should listen for events triggered by the SDK and adjust its behavior based on user consent status.

### Axeptio SDK Responsibilities:

1. **Consent Management UI**  
   The SDK is responsible for displaying the consent management interface.

2. **Storing and Managing User Consent**  
   It stores and manages consent choices across sessions.

3. **API Integration**  
   The SDK communicates user consent status through its API endpoints.

**Note:** The SDK does not manage ATT permissions. You must handle this separately as shown above.

***
# 5. Retrieving and Managing Stored Consents

To retrieve stored consent choices, use **UserDefaults** (iOS) or **SharedPreferences** (Android) with the `shared_preferences` package.

#### Dart Example

```dart
import 'package:shared_preferences/shared_preferences.dart';

// Access stored consents
SharedPreferences prefs = await SharedPreferences.getInstance();
String userConsent = prefs.getString('axeptio_consent');
```
***
# 6. Displaying the Consent Popup on Demand
If needed, you can display the consent popup manually by calling the following method:
```dart
axeptioSdk.showConsentScreen();
```
This is useful if you want to show the popup at a specific moment based on app flow or user behavior.

***
# 7. Sharing Consents with Web Views
For **publishers**, the SDK provides a feature to share the user's consent status with web views by appending the **Axeptio token** as a query parameter.
```dart
final token = await axeptioSdk.axeptioToken;
final url = await axeptioSdk.appendAxeptioTokenURL(  
  "https://myurl.com",  
  token,  
); 
// Will return: https://myurl.com?axeptio_token=[token]
```
This feature ensures that consent status is properly communicated across different parts of the application, including web content.
***

# 8. 🔄Clearing User Consent
If necessary, you can clear the user’s consent choices by invoking the following method:
```dart
axeptioSdk.clearConsent();
```
This will reset the consent status, allowing the user to be prompted for consent again.

***

# 9. Event Handling and Customization
The **Axeptio SDK** triggers various events that notify your app when the user takes specific actions related to consent. Use the `AxeptioEventListener` class to listen for these events and handle them as needed.
#### Example Event Listener Implementation:
```dart
var listener = AxeptioEventListener();

// Event triggered when the user closes the consent popup
listener.onPopupClosedEvent = () {
  // Retrieve consents from SharedPreferences/UserDefaults
  // Handle actions based on the user's consent preferences
};

// Event triggered when Google Consent Mode is updated
listener.onGoogleConsentModeUpdate = (consents) {
  // Handle updates to Google Consent Mode status
  // Perform specific actions based on updated consents
};

// Add and remove event listeners as necessary
var axeptioSdk = AxeptioSdk();
axeptioSdkPlugin.addEventListener(listener);
axeptioSdkPlugin.removeEventListener(listener);
```
***
By following this guide, you'll be able to integrate the Axeptio Flutter SDK effectively into your app, providing comprehensive consent management and ensuring compliance with privacy regulations.

For advanced configurations and further documentation, please refer to the official [Axeptio documentation](https://support.axeptio.eu/hc/en-gb).
We hope this guide helps you get started with the Axeptio Flutter SDK. Good luck with your integration, and thank you for choosing Axeptio!
