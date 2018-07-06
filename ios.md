---
title: iOS SDK
---

## Swift

* [Get Started](#ios-get-started)
1. [Cocoapods](#ios-cocoapods)
2. [Quickstart example](#ios-quickstart-example)
* [Configuration](#ios-configuration)
1. [Request permission for Location Services](#ios-request-permission-for-location-services)
2. [Add SDK to the project](#ios-add-sdk-to-the-project)
3. [Start/Stop location updates](#ios-start-stop-location-updates)
4. [Configure custom location manager](#ios-configure-custom-location-manager)
* [Tutorials](#ios-tutorials)
1. [Create a Mobile Device](#ios-create-a-mobile-device)
2. [Create a Pin Device](#ios-create-a-pin-device)
3. [Start/Stop Monitoring for a device](#ios-start-stop-monitoring-for-a-device)
4. [Apple Push Notification service](#ios-apple-push-notification-service)
5. [WebSocket](#ios-websocket)
6. [Polling](#ios-polling)
7. [Publish](#ios-publish)
8. [Subscribe](#ios-subscribe)
9. [Get Matches](#ios-get-matches)
10. [Local States request](#ios-local-states-request)
* [ChangeLog](#ios-changelog)
* [Supported Platform](#ios-supported-platform)


{: #ios-get-started}
### Get Started

To include Matchmore SDK in your project you need to use CocoaPods.

{: #ios-cocoapods}
#### CocoaPods
If you don't have CocoaPods installed on your computer, you'll need to execute this command in the terminal:
```
sudo gem install cocoapods
```

When installation is done, open **Terminal**, locate to your project folder and enter the following command:

`pod init`

It will create a file named Podfile in the main directory of the project.
Open the Podfile and add the following line under `use_frameworks`.

`pod 'Matchmore'`

Save the Podfile, and inside **Terminal** enter the following command:

`pod install`

{: #ios-quickstart-example}
#### Quickstart example
Setup application API key and world, get it for free from http://matchmore.io/.

Our quick sample shows you how to create a first device, a publication, a subscription and get the matches.
```swift
import Matchmore // <- Here is our Matchmore module import.
import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?

    var exampleMatchHandler: ExampleMatchHandler!

    func application(_: UIApplication, didFinishLaunchingWithOptions _: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        // Start Matchmore by providing the api-key.
        let config = MatchMoreConfig(apiKey: "YOUR_API_KEY")
        Matchmore.configure(config)

        // Create first device, publication and subscription. Please note that we're not caring about errors right now.
        Matchmore.startUsingMainDevice { result in
            guard case let .success(mainDevice) = result else { print(result.errorMessage ?? ""); return }
            print("Using device: \n\(mainDevice.encodeToJSON())")

            // Start Monitoring Matches
            self.exampleMatchHandler = ExampleMatchHandler { matches, _ in
                print("You've got new matches!!! \n\(matches.map { $0.encodeToJSON() })")
            }
            Matchmore.matchDelegates += self.exampleMatchHandler

            // Create New Publication
            Matchmore.createPublicationForMainDevice(publication: Publication(topic: "Test Topic", range: 20, duration: 100, properties: ["test": "true"]), completion: { result in
                switch result {
                case let .success(publication):
                    print("Pub was created: \n\(publication.encodeToJSON())")
                case let .failure(error):
                    print("\(String(describing: error?.message))")
                }
            })

            // Polling
            Matchmore.startPollingMatches(pollingTimeInterval: 5)
            self.createPollingSubscription()

            // Socket (requires world_id)
            Matchmore.startListeningForNewMatches()
            self.createSocketSubscription()

            // APNS (Subscriptions is being created after receiving device token)
            UIApplication.shared.registerForRemoteNotifications()

            Matchmore.startUpdatingLocation()
        }
        return true
    }

    // Subscriptions

    // Create a WebSocket subscription to Matchmore, and get your match notification via WebSocket
    func createSocketSubscription() {
        let subscription = Subscription(topic: "Test Topic", range: 20, duration: 100, selector: "test = true")
        subscription.pushers = ["ws"]
        Matchmore.createSubscriptionForMainDevice(subscription: subscription, completion: { result in
            switch result {
            case let .success(sub):
                print("Socket Sub was created \n\(sub.encodeToJSON())")
            case let .failure(error):
                print("\(String(describing: error?.message))")
            }
        })
    }

    // Create a polling subscription to Matchmore, and get your match notification via polling
    func createPollingSubscription() {
        let subscription = Subscription(topic: "Test Topic", range: 20, duration: 100, selector: "test = true")
        Matchmore.createSubscriptionForMainDevice(subscription: subscription, completion: { result in
            switch result {
            case let .success(sub):
                print("Polling Sub was created \n\(sub.encodeToJSON())")
            case let .failure(error):
                print("\(String(describing: error?.message))")
            }
        })
    }

    // Create a Apple push notification service subscription to Matchmore, and get your match notification via APNS
    func createApnsSubscription(_ deviceToken: String) {
        let subscription = Subscription(topic: "Test Topic", range: 20, duration: 100, selector: "test = true")
        subscription.pushers = ["apns://" + deviceToken]
        Matchmore.createSubscriptionForMainDevice(subscription: subscription, completion: { result in
            switch result {
            case let .success(sub):
                print("APNS Sub was created \n\(sub.encodeToJSON())")
            case let .failure(error):
                print("\(String(describing: error?.message))")
            }
        })
    }

    // MARK: - APNS
    // To add Apple push notification service you need to add these functions
    func application(_: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        // Convert token to string
        let deviceTokenString = deviceToken.reduce("", { $0 + String(format: "%02X", $1) })
        Matchmore.registerDeviceToken(deviceToken: deviceTokenString)

        createApnsSubscription(deviceTokenString)
    }

    func application(_: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable: Any]) {
        Matchmore.processPushNotification(pushNotification: userInfo)
    }
}
```

Define an object that's MatchDelegate implementing OnMatchClosure.
This object will handle the matches.

```swift
class ExampleMatchHandler: MatchDelegate {
    var onMatch: OnMatchClosure?
    init(_ onMatch: @escaping OnMatchClosure) {
        self.onMatch = onMatch
    }
}
```

{: #ios-configuration}
### Configuration

{: #ios-request-permission-for-location-services}
#### Request permission for Location Services
Depending on your needs, your app may require Location Services to work in the background, which means we need to set up Location Services usage description. This description will be shown to the user when asking them about allowing the app to access their location.

Users must grant permission for an app to access personal information, including the current location. Although people appreciate the convenience of using an app that has access to this information, they also expect to have control over their private data.

In the project navigator, find the *Info.plist* file, right click on it, and select “Open As”, “Source Code”. Then, inside the top-level <dict> section, add:

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

When opening app for the first time, the system will prompt `authorization alert` with the filled description. Users can decide wether to accept or reject the request permission.

`NSLocationWhenInUseUsageDescription` should describe how your app uses Location Services when it’s in use (or “in the foreground”).

`NSLocationAlwaysUsageDescription` should describe how your app uses Location Services both when in use, and in the background. This description is only for users of your app with iOS 10 or earlier. The user can agree to the “always” authorization, or disable Location Services in the app completely.

`NSLocationAlwaysAndWhenInUseUsageDescription` should describe how your app use Location Services both when in use, and in the background. This description is only for users of your app with iOS 11. The user can select between the “always” or “only when in use” authorizations, or disable Location Services in the app completely.


<div class="callout-block callout-success"><div class="icon-holder">*&nbsp;*{: .fa .fa-thumbs-up}
</div><div class="content">
{: .callout-title}
#### Good practice
Request permission at launch only when necessary for your app to function. Users won’t be bothered by this request if it’s obvious that your app depends on their personal information to operate. For example, an app might only request access to the current location when activating a location tracking feature.
</div></div>


{: #ios-add-sdk-to-the-project}
#### Add SDK to the project
Matchmore SDKs are very malleable. The SDK is configured by default to work for general cases. But it is possible to configure it according to your needs.
First set up Matchmore SDK Cloud credentials, this will allow the SDK to communicate with Matchmore Cloud on your behalf.

You can generate a token API-key for yourself on matchmore.io.
When you have your token, create your `MatchMoreConfig` with your API-key.

```swift
// MatchMoreConfig is a structure that defines all variables needed to configure MatchMore SDK.
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

{: #ios-start-stop-location-updates}
#### Start/Stop location updates

Both custom and default location managers can be started or stopped by simply calling:
```swift
// Start
Matchmore.startUpdatingLocation()

// Stop
Matchmore.stopUpdatingLocation()
```

{: #ios-configure-custom-location-manager}
#### Configure custom location manager

Location manager can be easily overwritten by custom configuration like this:
```swift
lazy var locationManager: CLLocationManager = {
        let locationManager = CLLocationManager()
        locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters
        locationManager.activityType = .fitness
        locationManager.pausesLocationUpdatesAutomatically = true
        locationManager.allowsBackgroundLocationUpdates = true
        return locationManager
    }()
```
Then make sure you inject your customer manager when configuring Matchmore:
```swift
let config = MatchMoreConfig(apiKey: "API_KEY",
              customLocationManager: self.locationManager)
```

{: #ios-tutorials}
### Tutorials

{: #ios-create-a-mobile-device}
#### Create a Mobile Device

If you need to create other mobile than device than main mobile device you can do so by:
```swift
Matchmore.mobileDevices.create(item: device) { (result) in
    switch result {
    case let .success(device):
        print("Created Mobile Device: \(device.id)")
    case let .failure(error):
        print(error?.localizedDescription)
    }
}
```

{: #ios-create-a-pin-device}
#### Create a Pin Device

To create new pin device you can simply call:
```swift
let location = Location(latitude: 46.519962, longitude: 6.633597, altitude: 0.0, horizontalAccuracy: 0.0, verticalAccuracy: 0.0)
let pin = PinDevice(name: "Pin Device", location: location)
Matchmore.createPinDevice(pinDevice: pin) { result in
    switch result {
    case let .success(device):
        print("Created Pin Device: \(device.id)")
    case let .failure(error):
        print(error?.localizedDescription)
    }
}
```

{: #ios-start-stop-monitoring-for-a-device}
#### Start/Stop Monitoring for device

When creating a main device, it is being monitored automatically. You can monitor any device.
For the following example, we monitor a pin device.

```swift
let location = Location(latitude: 46.519962, longitude: 6.633597, altitude: 0.0, horizontalAccuracy: 0.0, verticalAccuracy: 0.0)
let pin = PinDevice(name: "Pin Device", location: location)
Matchmore.createPinDevice(pinDevice: pin) { result in
    switch result {
    case let .success(device):
        print("Created Pin Device: \(device.id)")
        Matchmore.startMonitoringMatches(forDevice: device) // <- Start the monitoring of this pin device
    case let .failure(error):
        print(error?.localizedDescription)
    }
}
```

To stop the monitoring a device :
```swift
Matchmore.stopMonitoringMatches(forDevice: device) // <- Stop the monitoring of the instance in variable "device".
```

{: #ios-apple-push-notification-service}
##### Apple Push Notification service
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

###### Difference between Development and Production certificate

The choice of APNs host depends on which kinds of iOS app you wish to send push notifications to. There are two kinds of iOS app:

Published apps. These are published to the App Store and installed from there. These apps go through the usual App Store publication process, such as code-signing.

Development apps. These are “everything else”, such as apps installed from Xcode, or through private app publication systems.


<div class="callout-block callout-danger"><div class="icon-holder">*&nbsp;*{: .fa .fa-exclamation-triangle}
</div><div class="content">
{: .callout-title}
## Warning
Do not submit Apps to Apple with Development certificates.

</div></div>

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

Here is an example of a subscription getting matches with Apple Push Notification service :
```swift
// Create a Apple push notification service subscription to Matchmore, and get your match notification via APNS
func createApnsSubscription(_ deviceToken: String) {
    let subscription = Subscription(topic: "Test Topic", range: 20, duration: 100, selector: "test = true")
    subscription.pushers = ["apns://" + deviceToken] // <- Adding "apns://myDeviceToken" in the pushers parameter will activate the Apple Push Notification service
    Matchmore.createSubscriptionForMainDevice(subscription: subscription, completion: { result in
        switch result {
        case let .success(sub):
            print("APNS Sub was created \n\(sub.encodeToJSON())")
        case let .failure(error):
            print("\(String(describing: error?.message))")
        }
    })
}
```

{: #ios-websocket}
##### WebSocket
When it comes to deliver matches, Matchmore SDK uses WebSocket as well to provide alternative solutions to APNs.
By initializing this, you inform Matchmore Cloud service that you want to be notified via WebSocket.

```swift
    // MARK: - Socket
    func openSocketForMatches()
    func closeSocketForMatches()
```

Here is an example of a subscription getting matches with WebSocket :
```swift
// Create a WebSocket subscription to Matchmore, and get your match notification via WebSocket
func createSocketSubscription() {
    let subscription = Subscription(topic: "Test Topic", range: 20, duration: 100, selector: "test = true")
    subscription.pushers = ["ws"] // <- Adding "ws" in the pushers parameter will activate the WebSocket
    Matchmore.createSubscriptionForMainDevice(subscription: subscription, completion: { result in
        switch result {
        case let .success(sub):
            print("Socket Sub was created \n\(sub.encodeToJSON())")
        case let .failure(error):
            print("\(String(describing: error?.message))")
        }
    })
}
```

{: #ios-polling}
##### Polling
Use these functions to start or stop polling matches from Matchmore Cloud.

```swift
    // MARK: - Polling
    func startPollingMatches(pollingTimeInterval: 10)
    func stopPollingMatches()
```

Here is an example of a subscription getting matches with polling :
```swift
// Create a polling subscription to Matchmore, and get your match notification via polling
func createPollingSubscription() {
    let subscription = Subscription(topic: "Test Topic", range: 20, duration: 100, selector: "test = true")
    Matchmore.createSubscriptionForMainDevice(subscription: subscription, completion: { result in
        switch result {
        case let .success(sub):
            print("Polling Sub was created \n\(sub.encodeToJSON())")
        case let .failure(error):
            print("\(String(describing: error?.message))")
        }
    })
}
```

{: #ios-publish}
#### Publish
```swift
// Create New Publication
var pub = Publication(topic: "Test Topic", range: 100, duration: 30, properties: ["test": "true", "price": 199])
Matchmore.createPublicationForMainDevice(publication: pub, completion: { result in
    switch result {
    case let .success(publication):
        print("Pub was created: \n\(publication.encodeToJSON())")
    case let .failure(error):
        print("\(String(describing: error?.message))")
    }
})
```

{: #ios-subscribe}
#### Subscribe
```swift
// Create a WebSocket subscription to Matchmore, and get your match notification via WebSocket
func createSocketSubscription() {
    let subscription = Subscription(topic: "Test Topic", range: 100, duration: 30, selector: "test = true")
    subscription.pushers = ["ws"] // <- Adding "ws" in the pushers parameter will activate the WebSocket
    Matchmore.createSubscriptionForMainDevice(subscription: subscription, completion: { result in
        switch result {
        case let .success(sub):
            print("Socket Sub was created \n\(sub.encodeToJSON())")
        case let .failure(error):
            print("\(String(describing: error?.message))")
        }
    })
}

// Create a polling subscription to Matchmore, and get your match notification via polling
func createPollingSubscription() {
    let subscription = Subscription(topic: "Test Topic", range: 20, duration: 100, selector: "test = true")
    Matchmore.createSubscriptionForMainDevice(subscription: subscription, completion: { result in
        switch result {
        case let .success(sub):
            print("Polling Sub was created \n\(sub.encodeToJSON())")
        case let .failure(error):
            print("\(String(describing: error?.message))")
        }
    })
}

// Create a Apple push notification service subscription to Matchmore, and get your match notification via APNS
func createApnsSubscription(_ deviceToken: String) {
    let subscription = Subscription(topic: "Test Topic", range: 20, duration: 100, selector: "test = true")
    subscription.pushers = ["apns://" + deviceToken] // <- Adding "apns://myDeviceToken" in the pushers parameter will activate the Apple Push Notification service
    Matchmore.createSubscriptionForMainDevice(subscription: subscription, completion: { result in
        switch result {
        case let .success(sub):
            print("APNS Sub was created \n\(sub.encodeToJSON())")
        case let .failure(error):
            print("\(String(describing: error?.message))")
        }
    })
}
```

{: #ios-get-matches}
#### Get Matches
```swift
//you can call to get matches at your leisure, if you don't want to use the monitors. In the end the monitors are calling exactly this method for you.
Matchmore.refreshMatches()
```

{: #ios-local-states-request}
#### Local States request (Create, Find, FindAll, Delete and DeleteAll)
```swift
// You can access locally cached subscriptions, publications and devices
Matchmore.subscriptions
Matchmore.publications
Matchmore.mainDevice
Matchmore.pinDevices
Matchmore.knownBeacons

// you can delete also your sub, pub and device by calling:
Matchmore.subscriptions.delete(item: sub, completion: { error in
                print(error?.message)
            })
Matchmore.publications.delete(item: pub, completion: { error in
                print(error?.message)
            })
Matchmore.pinDevices.delete(item: pin, completion: { (error) in
                print(error?.message)
            })
Matchmore.mobileDevices.delete(item: mobile, completion: { (error) in
                print(error?.message)
            })
```
