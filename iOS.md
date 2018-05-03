---
title: iOS integration & configuration
---

<!--
- Installation
- Start Matchmore
- Requesting permission for location services
- Apple Push Notification service
- Web socket iOS
- Polling iOS
-->
### installation
#### Carthage
#### other installations ?
#### Inject MatchmoreSDK with CocoaPods
If you don't have CocoaPods installed on your computer, you'll need to execute this command in the terminal:
```
sudo gem install cocoapods
```

When installation is done, open **Terminal**, locate to your project folder and enter the following command:

`pod init`

It will create a file named Podfile in the main directory of the project.
Open the Podfile and add the following line under `use_frameworks`.

`pod 'AlpsSDK', '~> 0.6'`

See example below.

```
target 'myApp' do
  # Comment the next line if you're not using Swift ...
  use_frameworks!

  pod 'AlpsSDK', '~> 0.6'
  ...
```

Save the Podfile, and inside **Terminal** enter the following command:

`pod install`


{: ##start-matchmore}
#### Start Matchmore
MatchmoreSDK is very malleable. It is configured by default to work for general cases. But it is possible to configure it according to your needs.
First set up MatchmoreSDK Cloud credentials, this will allow the SDK to communicate with Matchmore Cloud on your behalf.

You can generate a token API-key for yourself on matchmore.io.
When you have your token, create your `MatchMoreConfig` with your API-key.

```swift
/// MatchMoreConfig is a structure that defines all variables needed to configure MatchMore SDK.
public struct MatchMoreConfig {
    let apiKey: String
    let serverUrl: String
    let customLocationManager: CLLocationManager?

    public init(apiKey: String,
                serverUrl: String = "https://api.matchmore.io/v5",
                customLocationManager: CLLocationManager? = nil) {
        self.apiKey = apiKey
        self.serverUrl = serverUrl
        self.customLocationManager = customLocationManager
    }
}
```

Starting with a basic setup. Use MatchmoreSDK with default configuration.

```swift
// Basic setup
let config = MatchMoreConfig(apiKey: "YOUR_API_KEY") // create your own app at https://www.matchmore.io
MatchMore.configure(config)
```


#### Custom Integration
You can also customize MatchmoreSDK to meet other needs than those provided by default.
In the example below, simply inject the custom location manager into the MatchmoreSDK.

```swift
// Custom Location Manager
let locationManager = CLLocationManager()
locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters
locationManager.pausesLocationUpdatesAutomatically = true
locationManager.allowsBackgroundLocationUpdates = true
locationManager.requestAlwaysAuthorization()

// Setup MatchmoreSDK
let config = MatchMoreConfig(apiKey: "YOUR_API_KEY", customLocationManager: locationManager) // create your own app at https://www.matchmore.io
MatchMore.configure(config)
```


#### Requesting permission for Location Services
Depending on your needs, your app may require Location Services to work in the background, which means we need to set up Location Services usage description. This description will be shown to the user when asking them about allowing the app to access their location.

Users must grant permission for an app to access personal information, including the current location. Although people appreciate the convenience of using an app that has access to this information, they also expect to have control over their private data.

In the project navigator, find the Info.plist file, right click on it, and select “Open As”, “Source Code”. Then, inside the top-level <dict> section, add:

```XML
<key>NSLocationWhenInUseUsageDescription</key>
<string>MyApp will show you detected nearby devices.</string>

<!-- iOS 10 or earlier -->
<key>NSLocationAlwaysUsageDescription</key>
<string>MyApp will show you detected nearby devices, and alert you via
notifications if you don't have the app open.</string>

<!-- iOS 11 -->
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>MyApp will show you detected nearby devices in the app. With the "always" option,
we can also alert you via notifications if you don't have the app open.</string>
```
Why the three keys and what do they mean?

NSLocationWhenInUseUsageDescription should describe how your app uses Location Services when it’s in use (or “in the foreground”).

NSLocationAlwaysUsageDescription should describe how your app uses Location Services both when in use, and in the background. This description is only for users of your app with iOS 10 or earlier. The user can agree to the “always” authorization, or disable Location Services in the app completely.

NSLocationAlwaysAndWhenInUseUsageDescription should describe how your app use Location Services both when in use, and in the background. This description is only for users of your app with iOS 11. The user can select between the “always” or “only when in use” authorizations, or disable Location Services in the app completely.

When opening app for the first time, the system will prompt `authorization alert` with the filled description. Users can decide wether to accept or reject the request permission.

*GOOD PRACTICE* Request permission at launch only when necessary for your app to function. Users won’t be bothered by this request if it’s obvious that your app depends on their personal information to operate. For example, an app might only request access to the current location when activating a location tracking feature.


#### Apple Push Notification service
We use Apple Push Notification service (APNs). It allows app developers to propagate information via notifications from servers to iOS, tvOS and macOS devices.

In order to get the certificates and upload them to our portal, `**A membership in the Apple iOS developer program is required.**`
1. Enabling the Push Notification Service via Xcode

First, go to App Settings -> General and change Bundle Identifier to something unique.

Right below this, select your development Team. **This must be a paid developer account**.

After that, you need to create an App ID in your developer account that has the push notification entitlement enabled. Xcode has a simple way to do this. Go to App Settings -> Capabilities and flip the switch for Push Notifications to On.

Note: If any issues occur, visit the Apple Developer Center. You may simply need to agree to a new developer license.

At this point, you should have the App ID created and the push notifications entitlement should be added to your project. You can log into the Apple Developer Center and verify this.

2. Enable Remote Push Notifications on your App ID

Sign in to your account in the **Apple developer center**, and then go to Certificates, Identifiers & Profiles

Go to Identifiers -> App IDs. If you followed previous instructions, you should be able to see the App ID of your application( create it if you don't see it). Click on your application and ensure that the Push Notifications service is enabled.

Click on the Development or Production certificate button and follow the steps. We recommend you for a Production push certificate. It works for most general cases and is the required certificate for Apple store.

##### Difference between Development and Production certificate

The choice of APNs host depends on which kinds of iOS app you wish to send push notifications to. There are two kinds of iOS app:

    Published apps. These are published to the App Store and installed from there. These apps go through the usual App Store publication process, such as code-signing.

    Development apps. These are “everything else”, such as apps installed from Xcode, or through private app publication systems.

**Warning**
Do not submit Apps to Apple with Development certificates.


3. Exporting certificates

After completing the steps to generate a Development SSL or Production SSL certificat and downloading the certificate.

From Finder, double-click the ~/certificate.cer file, or run open ~/certificate.cer. This will open Keychain Access. The certificate should now be added to the list under “My Certificates”.

Right click on the certificate, and select "Export ....". Select the p12 format and create a password for the file, do not lose this password as you will need it a next step.
**For now : Leave this password blank.**

Upload the generated certificate to Matchmore Portal.

4. Enable Push Notification in Application

Add the following lines to your appDelegate.

```swift
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    // Convert token to string
    let deviceTokenString = deviceToken.reduce("", {$0 + String(format: "%02X", $1)})
    MatchMore.registerDeviceToken(deviceToken: deviceTokenString)
    createApnsSubscription()
}

func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any]) {
    MatchMore.processPushNotification(pushNotification: userInfo)
}
```

You can now start sending notifications to your users.

{: #web-socket-ios}
#### Web Socket
When it comes to deliver matches, Matchmore SDK uses Web socket as well to provide alternative solutions to APNs.
By initializing this, you inform Matchmore Cloud service that you want to be notified via web socket.

```swift
    // MARK: - Socket
    func openSocketForMatches()
    func closeSocketForMatches()
```

{: #polling-ios}
#### Polling
Use these functions to start or stop polling matches from Matchmore Cloud.

```swift
    // MARK: - Polling
    func startPollingMatches(pollingTimeInterval: TimeInterval)
    func stopPollingMatches()
```
