# Home
## Matchmore
Documentation

As the fast-growing world of mobile connected objects computing era is becoming a reality, Matchmore aims at becoming the leading cloud-based software platform supporting the creation of highly dynamic proximity-based applications. At the heart of our vision is the notion of geomatching, which builds on the location-based publish/subscribe communication model.

### Description

Our goal is to provide tools that dramatically simplify and accelerate the development, testing and deployment of rich application scenarios based on **multiple moving and connected objects**, making such applications the new low hanging fruits of the mobile app industry.

Get familiar with the Matchmore products and explore their features:

**links to our products opens quickstart of || ALPS || PHOMO**

### Understanding Matchmore
Matchmore helps you model any geolocated or proximity-based applications, by taking advantage of advanced location-based publish/subscribe (ALPS) model, create any type of interactions for connected objects from smartphones to any sensors and everything in between.

#### In-depth Publish/subscribe
The purpose of Publish/Subscribe model is to create communication between several computers, applications, and users. Entities interested in a specific topic subscribe to the topic and whenever there is a message on this topic, they will receive it. Content-based filtering is an additional layer, namely selector, allowing subscribers to filter incoming messages based on the message’s content. On the other hand, publishers/senders are able to send messages without any knowledge of the future receivers, ensuring anonymity. Concerning content-based filtering, publishers are able to customize their messages with the additional layer, namely properties. For instance, a publication and a subscription were issued on the same topic and have compatible properties and the evaluation of the selector against those properties returns a true value, so there is a content correspondence which leads to a message delivery.
**/OR SHORTER/**

The publish/subscribe model is a message-oriented paradigm in which, publishers can publish anonymously on a topic and subscribers can receive messages on that topic. The additional layer, namely selector, allows subscribers to filter incoming messages based on the message’s content.

#### Advanced Location based Publish/Subscribe
ALPS is designed to simplify software developer’s life. It is an extension of the Publish-Subscribe model adding user context while remaining message-oriented. Publications and subscriptions are extended with the notion of geographical zone. The zone is defined as a circle with a center at the given location and a range around that location.
Publications and subscriptions which are associated with a mobile device, e.g. user’s mobile phone, potentially follow the movements of the user carrying the device and therefore change their associated location. The crossing of the range and corresponding topic and content lead to the delivery of the message, in fact, a **match**.

To facilitate the mobile application development, ALPS is provided in enhanced Software Development Kits (SDK).

So what exactly can you do with Matchmore?

### Geomatching (Matching Moving Content)
ALPS enables you to notify the user whenever there’s someone/something they could be interested in close by. Create proximity detection easily on web or mobile by using the advanced location-based publish/subscribe paradigm.

### Create Smart Geofences
Geofences are used to define special areas in the real world, that you are particularly interested in.
With smart geofences, you can go further in the filtering process. It can be used for analytics uses: count how many app users based on their preference enter a specific area, thanks to the topic and content filtering. Or for triggering content inside the mobile or externally depending on user's attached pieces of information. Works indoor (beacons) and outdoor (GPS).

### Smart activation with IoT
Activate a connected device when you approach it (a camera by example).

### Geochat
Broadcast anonymous message to people around you.

### Smart checkpoints
Create virtual gates or points to control the passage of users. A trail is validated if the user has passed by every point related to the trail.

### And many other uses that we didn't think of yet..!
We have a sample app where we grouped a lot of interesting uses, here is a link to the Github.
ALPS paradigm is applicable to many domains and eases the modelization of any geolocated applications.

### Some links to help // (Examples of links : contact us, examples, tutorials)
### Redirection // (What next, OR recently added)
### Questions ?
We're always happy to help with code or other questions you might have! Search our documentation, contact support, or connect with our sales team.

**/GENERAL IMPRESSION AT THE END OF THIS PAGE/ The reader should understand Matchmore briefly and should be able to find his next step.**

# Proximity Detection or ALPS

## Quickstart
Quickly create proximity detection for any type of devices.

1. Create a device to **feature proximity detection**
MatchmoreSDK can be used in different ways, but the most common way is to have a main device. Usually, this main device is the device on which the app is running.

**/SWIFT, KOTLIN or/AND JAVA, UNITY and JS/**
```swift
// a. Create the main device
        MatchMore.startUsingMainDevice { result in
            // b. Unwrap the result
            guard case .success(let mainDevice) = result else { print(result.errorMessage ?? ""); return }
            print("🏔 Using device: 🏔\n\(mainDevice.encodeToJSON())")

            // c. Start getting matches
            MatchMore.startPollingMatches()
            // d. Start location track in Matchmore
            MatchMore.startUpdatingLocation()
        }
```

* a. Use the wrapper MatchMore and call the method startUsingMainDevice() to start the registration of the app running device in Matchmore service.
* b. MatchMore SDK's methods are generally having an asynchronous callback. To handle that, you need to declare closure. `result` is an enum, you need to unwrap it. When it is a success, the callback returns the object created in Matchmore's side, else when it is a failure, the callback is an error containing a message that describes what failed.
* c. Start polling the matches.
* d. Start informing Matchmore service that the user's main device is moving.

2. Create a publication to **broadcast the presence of a device**
When a device is a publisher, it is as if it broadcasts its presence. The publications can contain a lot of information, all this information is grouped in the `properties` array.
N.B .: A device can be both publisher and subscriber at the same time.

Let's create a publication.
**/SWIFT, KOTLIN, JAVA, UNITY and JS/**
```swift
        // a. Create a Publication, set topic and properties
        let publication = Publication(topic: "test", range: 100, duration: 60, properties: ["band": "Metallica",
        "tags": ["rock", "metal"],
        "price": 400,
        "currency": "CHF",
        "free_parking": true])
        // b. Create a Publication for Main Device
        MatchMore.createPublicationForMainDevice(publication: publication, completion: { result in
            switch result {
            case .success(let publication):
                print("🏔 Pub was created: 🏔\n\(publication.encodeToJSON())")
            case .failure(let error):
                print("🌋 \(String(describing: error?.message)) 🌋")
            }
        })
    }
```
* a. Create a publication and fulfill the required parameters.
- The topic is our first filter. In order to match, both a publication and a subscription need to be created on the same unique topic.
Here, the topic is *test*. Every publication that is set on topic *test* will concern only the *test* features. Use topic to manage different channels of matches.
- You can select the size of the zone around any publications/subscriptions, it is defined as a circle with a center at the given location and a range around that location. The range is defined in meters.
- You can also set the duration of your publications. The duration is defined in seconds.
- Now, you can set some properties to transmit either supplement information or to allow finer filtering for the subscribers of your topic.
The following data types are allowed in `Properties`: `String`, `Int`, `Set` and `Boolean`.
* b. Use the wrapper MatchMore and call the method createPublicationForMainDevice() to send the creation request to Matchmore service.
N.B.: Don't forget we need to handle two cases, in case of success we retrieve the created object, and in case of failure, we retrieve an error.
3. Create a subscription to **start discovering near devices**
Subscribers are notified when matches occur.
Please be aware that **publishers** are NOT notified of matches occurrence, if you want your publishers to be notified, he should be subscribing as well.
```swift
        // a. Create a Subscription, set topic and selector.
        let subscription = Subscription(topic: "test", range: 1, duration: 60, selector: "band <> 'Ramstein' AND price < 500")
        // b. Create a Subscription for Main Device
        MatchMore.createSubscriptionForMainDevice(subscription: subscription, completion: { result in
            switch result {
            case .success(let sub):
                print("🏔 Socket Sub was created 🏔\n\(sub.encodeToJSON())")
            case .failure(let error):
                print("🌋 \(String(describing: error?.message)) 🌋")
            }
        })
    }
```

Create a subscription to await matches on the `test` topic.
* Remember the topic is the first filter you can play with.
* As for Publications, you can set the range and the duration you want for your subscriptions.
* The selector can be used to granularly filter the publication's properties. In this example, we want `key: band`'s value does not equal to `Ramstein` AND `key: price`'s value has to be lower than `500`. Since this query returns true, we will get a match between the previous publication and this subscription.
* Use the wrapper MatchMore and call the method createSubscriptionForMainDevice() to send the creation request to Matchmore service.
N.B.: Don't forget we need to handle two cases, in case of success we retrieve the created object, and in case of failure, we retrieve an error.
4. When matches occur subscribers are notified of the presence of the publisher
When matches occur, all classes conform to `MatchDelegate` protocol are notified.

Define an object that's `MatchDelegate` implementing OnMatchClosure.

```swift
class ExampleMatchHandler: MatchDelegate {
    var onMatch: OnMatchClosure?
    init(_ onMatch: @escaping OnMatchClosure) {
        self.onMatch = onMatch
    }
}
```

In this example we make class ViewController conform to the MatchDelegate protocol.
Also, you need to add var onMatch which is a closure.
```swift
class ViewController: UIViewController, UIPickerViewDelegate, UIPickerViewDataSource, MatchDelegate {
var onMatch: OnMatchClosure?
```

Next, implement how your matches should be handled:

```swift
       // Handle Matches
       self.onMatch = { [weak self] matches, device in
           // a. Unwrap properties
           guard let properties = mostRecentDate?.publication?.properties else {
               print("No properties.")
               return
           }
           guard let color = properties["color"] as? String else {return}

           // b. Do something
       }
       // c. Add View as a supplement match delegate to Matchmore
       MatchMore.matchDelegates += self
   }
```

You'll create the delegate function for the onMatch listener. onMatchClosure are triggered every time you get a new match.
The returned callback is composed of an array of hundred last matches, plus the concerned device.

* a. Safely unwrap the value you'll use.
* b. Use the match information as you need
* c. Earlier, we have made ViewController conform to MatchDelegate protocol. In order to be informed of every match, you'll need to add ViewController to Matchmore wrapper match delegates list.
## IOS (Configuration(Info.plist, CocoaPods, API-Key, MainDevice, Pub, Sub, Match, Start updating location))
## Versioning

SDK is written using Swift 4.

MatchmoreSDK requires iOS 9+.

### Inject MatchmoreSDK with CocoaPods
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

### Requesting permission for Location Services
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

### Set up Matchmore Cloud Credentials
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
let config = MatchMoreConfig(apiKey: "eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiJ9.eyJpc3MiOiJhbHBzIiwic3ViIjoiMTc3NjdhN2UtNzU0OC00ZTk5LWEwMGMtYTFkYTM2NmIwNGFmIiwiYXVkIjpbIlB1YmxpYyJdLCJuYmYiOjE1MjA0MjQyMDQsImlhdCI6MTUyMDQyNDIwNCwianRpIjoiMSJ9.ayQMcG6jPmVX4eVfqRwcdrwAAOblZPXHVV8WuZSG7m8dWCr65lgvwnIxwqOBg3Oco2aUiKuNaBWllmfpKawaug") // create your own app at https://www.matchmore.io
MatchMore.configure(config)
```

### Custom Integration
You can also custom MatchmoreSDK to fit different needs other than provided by default.
In the example below, simply inject the custom location manager inside of MatchmoreSDK.

```swift
// Custom Location Manager
let locationManager = CLLocationManager()
locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters
locationManager.pausesLocationUpdatesAutomatically = true
locationManager.allowsBackgroundLocationUpdates = true
locationManager.requestAlwaysAuthorization()

// Setup MatchmoreSDK
let config = MatchMoreConfig(apiKey: "eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiJ9.eyJpc3MiOiJhbHBzIiwic3ViIjoiMTc3NjdhN2UtNzU0OC00ZTk5LWEwMGMtYTFkYTM2NmIwNGFmIiwiYXVkIjpbIlB1YmxpYyJdLCJuYmYiOjE1MjA0MjQyMDQsImlhdCI6MTUyMDQyNDIwNCwianRpIjoiMSJ9.ayQMcG6jPmVX4eVfqRwcdrwAAOblZPXHVV8WuZSG7m8dWCr65lgvwnIxwqOBg3Oco2aUiKuNaBWllmfpKawaug", customLocationManager: locationManager) // create your own app at https://www.matchmore.io
MatchMore.configure(config)
```
### Apple Push Notification service
We use Apple Push Notification service (APNs), it allows app developers to propagate information via notifications from servers to iOS, tvOS and macOS devices.

In order to get the certificates and upload them to our portal, `**A membership in the Apple iOS developer program is required.**`
1. Enabling the Push Notification Service via Xcode

First, go to App Settings -> General and change Bundle Identifier to something unique.

Right below this, select your development Team. **This must be a paid developer account**.

After that, you need to create an App ID in your developer account that has the push notification entitlement enabled. Xcode has a simple way to do this. Go to App Settings -> Capabilities and flip the switch for Push Notifications to On.

![apns capabilities switch](https://github.com/matchmore/alps-ios-sdk/blob/master/assets/apns1.png)

Note: If any issues occur, visit the Apple Developer Center. You may simply need to agree to a new developer license.

At this point, you should have the App ID created and the push notifications entitlement should be added to your project. You can log into the Apple Developer Center and verify this.

![verification apns activated](https://github.com/matchmore/alps-ios-sdk/blob/master/assets/apns2.png)

2. Enable Remote Push Notifications on your App ID

Sign in to your account in the **Apple developer center**, and then go to Certificates, Identifiers & Profiles

Go to Identifiers -> App IDs. If you followed previous instructions, you should be able to see the App ID of your application( create it if you don't see it). Click on your application and ensure that the Push Notifications service is enabled.

![apns capabilities switch](https://github.com/matchmore/alps-ios-sdk/blob/master/assets/apns3.png)

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

You can now start sending notifications to your users using Matchmore SDK.
### Web Socket
When it comes to deliver matches, ALPS uses Web socket as well to provide alternative solutions to APNs.

```swift
    // MARK: - Socket
    func openSocketForMatches() {
        if socket != nil { return }
        guard let deviceId = monitoredDevices.first?.id else { return }
        let worldId = MatchMore.config.apiKey.getWorldIdFromToken()
        var url = MatchMore.config.serverUrl
        url = url.replacingOccurrences(of: "https://", with: "")
        url = url.replacingOccurrences(of: "http://", with: "")
        url = url.replacingOccurrences(of: "/v5", with: "")
        let request = URLRequest(url: URL(string: "ws://\(url)/pusher/v5/ws/\(deviceId)")!)
        socket = WebSocket(request: request, protocols: ["api-key", worldId])
        socket?.disableSSLCertValidation = true
        socket?.onText = { text in
            if text != "ping" && text != "" && text != "pong" { // empty string or "ping" just keeps connection alive
                self.getMatches()
            }
        }
        socket?.onDisconnect = { error in
            self.socket?.connect()
        }
        socket?.onPong = { _ in
            self.socket?.write(ping: "ping".data(using: .utf8)!)
        }
        socket?.connect()
    }

    func closeSocketForMatches() {
        socket?.disconnect()
        socket = nil
    }
```
### Polling
```swift
    // MARK: - Polling
    func startPollingMatches(pollingTimeInterval: TimeInterval) {
        if timer != nil { return }
        timer = Timer.scheduledTimer(timeInterval: pollingTimeInterval, target: self, selector: #selector(getMatches), userInfo: nil, repeats: true)
    }

    func stopPollingMatches() {
        timer?.invalidate()
        timer = nil
    }
```
## Android
### Standard Integration
### Custom Integration
### Google Firebase Cloud Notification
We use Apple Push Notification service (APNs) which is the main remote notification service provided by Apple. APNs allows app developers to propagate information via notifications from servers to iOS, tvOS and macOS devices.
### Web Socket
When it comes to deliver matches, ALPS uses Web socket as well to provide alternative solutions to APNs.
### Polling
## Unity 3D
### Standard Integration
### Custom Integration
## JS
### Standard Integration
### Custom Integration

## Devices
A device might be either virtual like a pin device or physical like a mobile device.
Devices may issue publications and subscriptions at any time; it may also cancel publications and subscriptions issued previously. Publications and subscriptions do have a definable, finite duration, after which they are deleted from the Matchmore cloud service and don’t participate anymore in the matching process.

### Mobile
A mobile device is one that potentially moves together with its user and therefore has a geographical location associated with it. A mobile device is typically a location-aware smartphone, which knows its location thanks to GPS or to some other means like cell tower triangulation, etc.

### Pin
A pin device is one that has geographical location associated with it but is not represented by any object in the physical world; usually it’s location doesn’t change frequently if at all.

#### Using Pin Device

### Beacon
Beacons are high-tech tools that repeatedly broadcast a single signal under the form of advertising packet. Other devices interact with beacons through bluetooth and receive an advertising packet which consist of different letters and numbers. With the information received through the packet, devices like smartphones know how close they are to a specific beacon. The main purpose behind beacons is to improve indoor location. When developers know how close they are to this specific location, thanks to beacons, they can do something useful with this information.
#### Beacons standard
The iBeacon is the standard defined by Apple. There is also other beacons standard like Eddystone by Google or AltBeacons by Kontakt.io. Basically, the content of the advertising packet could vary slightly from one standard to another, but the communication protocol (bluetooth) remains the same. As a consequence, most beacons on the market support at least the iBeacon standard and the Eddystone standard.

## Publication
A publication is similar to a Publish-Subscribe model publication extended with the notion of a geographical zone. The zone is defined as circle with a center at the given location and a range around that location. Publications which are associated with a mobile device, e.g. user’s mobile phone, potentially follow the movements of the user carrying the device and therefore change their associated location.

The following data types are allowed in Properties:  `String`, `Int`, `Set` and `Boolean`.

## Subscription
A subscription is similar to a Publish- Subscribe subscription extended with the notion of geographical zone. The zone is defined as circle with a center at the given location and a range around that location. Subscriptions which are associated with a mobile device, e.g. user’s mobile phone, potentially follow the movements of the user carrying the device and therefore change their associated location.

SQL Comparison Operators
```SQL
=    Equal to    
>    Greater than    
<    Less than    
>=    Greater than or equal to    
<=    Less than or equal to    
<>    Not equal to
```

SQL Logical Operators
```SQL
ALL    TRUE if all of the subquery values meet the condition    
AND    TRUE if all the conditions separated by AND is TRUE    
ANY    TRUE if any of the subquery values meet the condition    
BETWEEN    TRUE if the operand is within the range of comparisons    
IN    TRUE if the operand is equal to one of a list of expressions    
LIKE    TRUE if the operand matches a pattern    
NOT    Displays a record if the condition(s) is NOT TRUE    
OR    TRUE if any of the conditions separated by OR is TRUE    
```

## Match
A match is equal to a Publish-Subscribe message delivery, it occurs when both of the two conditions hold: Content correspondence and context match. Content correspondence happens when publishers publish on content in which subscribers subscribe. Context matching occurs when for instance the subscription zone overlaps with the publication zone while both of pub/sub are still active (Not exceeded duration).
## Best Practices

# Dashboard
Dashboard allows the developers to register their application and services, monitor the costs and interact with their applications.
## Overview
## Apps
## Beacons
## Tools
## Billing
## Phomo

# Account
## Your account
