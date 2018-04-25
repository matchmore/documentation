---
title: Proximity Detection
sections:
  - Quickstart
  - Modeling
  - iOS
  - Android
  - Unity
  - JavaScript
---

{: #quickstart}
### Quickstart
Quickly create proximity detection for any types of `Device`.

Creating proximity detection using Matchmore is a four steps process:
1. Create a device to **feature proximity detection**
2. Create a publication to **broadcast the presence of a device**
3. Create a subscription to **start discovering near devices**
4. When subscribers and publishers are within range of each other, there is a `Match`

The [devices](#Device) are core models for proximity detection. A `Device` defines an object that is either virtual or physical which you can find useful when it is proximity detected. A device without publication or subscription is not detectable nor detecting, you need to attach `Publication` or `Subscription` on a `Device` in order to start either **broadcast presence** or **discover nearby devices**.

`Match` is the event object to notify a proximity detection based on the encounter of two different devices.

#### STEP 1: Create a device to **feature proximity detection**
Matchmore relies on different sensors to provide the proximity detection; such as GPS, Bluetooth, and more. Each `Device` is unique and offers distinct possibilities:
* `Mobile Device` is based on a constant `Location` update, thanks to GPS,
* `Beacon Device` is made for indoor and outdoor location services. It relies on Bluetooth technology,
* `Pin Device` is non-physical and at a fixed geographical `Location`.
* More types of `Device` will be added in the future.

Matchmore SDK can be used in different ways, but the most common way is to have a `Main Device`. Usually, this `Main Device` is the device on which the app is running, typically it represents your actual users; Smartphones (mobile app) or Browser (web app).

The following code demonstrates how to create the main device:
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
* b. MatchMore SDK's methods are generally having an asynchronous callback. To handle that, you need to declare a closure. `result` is an enum, you need to unwrap it. When it is a success, the callback returns the object created in Matchmore's side, else when it is a failure, the callback is an error containing a message that describes what failed.

In case of success, Matchmore returns a new device object with all of the relevant details:
```JSON
{
    "id": "84579fd3-58da-433f-a776-ed01c5c3f992",
    "createdAt": 1524214702782,
    "name": "My iphone",
    "platform": "iOs 11",
    "deviceToken": "some-apn-token-for-push",
    "location": {
        "latitude": 46.445015,
        "longitude": 6.908000,
        "altitude": 452
    }
}
```

Once you create the device, store the id value in your own database for later reference (presumably with the user's email address).
Devices have a unique ID that is generated automatically upon device creation.

When creating a device through the API, the response includes an ID, which you can use to attach `Publication` or `Subscription`. Provide this ID in your Matchmore requests to attach a publication/subscription to an existing device, pass the ID of the desired device as the deviceId value of the publication/subscription parameter.
* c. When using our SDK, polling the `Match` in Matchmore cloud service is that easy, just one line of code.
* d. Again, using our SDK will discharge you of many tasks. This single line of code starts informing Matchmore service of the user's main device updates location.

#### STEP 2: Create a publication to **broadcast the presence of a device**
When a `Device` has a `Publication` attached, it is as if it broadcasts its presence. You can see a publication as a message, you might store metadata, such as a phone number, a name, or any relevant information, on the publication object. Publications can contain customized information destined for future `Match` receivers. The metadata are grouped in the `properties` parameter.
You can attach as many `Publication` and `Subscription` as you want to a `Device`. It means a device can be both publisher and subscriber at the same time. You may need to define just one publication/subscription or several hundred, depending on how many services you offer.
**Please be aware that publishers** are NOT notified of matches occurrence, if you want your publishers to be notified, he must be subscribing as well.

This code creates a publication via the SDK and this publication will be directly attached to the main device:
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
- The topic is our first filter. In order to match, both a `Publication` and a `Subscription` need to be created on the same unique topic.
Here, the topic is *test*. Every publication that is set on topic *test* will concern only the *test* features. Use topic to manage different channels of matches.
- You can select the size of the zone around any publications/subscriptions, it is defined as a circle with a center at the given location and a range around that location. The range is defined in meters.
- You can also set the duration of your publications. The duration is defined in seconds.
- Now, you can set some properties to transmit supplement metadata to allow finer filtering for the subscribers of your topic.
The following data types are allowed in `Properties` parameters: `String`, `Int`, `Set` and `Boolean`.
* b. Use the MatchMore SDK and call the method createPublicationForMainDevice() to send the creation request to Matchmore service.
N.B.: Don't forget we need to handle two cases, in case of success we retrieve the created object, and in case of failure, we retrieve an error.
In case of success, Matchmore returns a new `Publication` object with all the relevant details:
```JSON
{
  "id": "c62b5a4f-ce4a-4249-a1fa-33235732ce13",
  "createdAt": 1524212756568,
  "deviceId": "cc8ec61f-913c-4e81-b959-01153f8eb46d",
  "topic": "test",
  "range": 100,
  "duration": 60,
  "properties": {
    "band": "Metallica",
    "tags": [
      "rock",
      "metal"
    ],
    "price": 400,
    "currency": "CHF",
    "free_parking": true
  }
}
```
Once you create the publication, store the id value in your own database for later reference (presumably with the user's deviceId).

#### STEP 3: Create a subscription to **start discovering near devices**
When a `Match` occurs, every **subscriber** receive a message based on their topic and content selection. With the selector parameter, you can filter publications based on the metadata contained within them to discover only the devices that might interest you.

Matchmore offers the ability to simultaneously subscribe a device to multiple topics. This provides supplement channel for easily include your features in flexible ways. For example, if your business offers multiple features or services based on proximity detection, each could be represented by its own topic and the same device could be subscribed to more than one topic. Imagine an app that has two features: geochatting (Broadcast messages to nearby users) and proximity detection (discover how many users are nearby).
You can create the features on two different topics for example, "geochat" in regards of geochatting and "detection" in regards of proximity detection to fully separate both features.

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

* a. Create a subscription to await matches on the `test` topic.
- Remember the topic is the first filter you can play with.
- As for Publications, you can set the desired range and duration for subscriptions.
- The selector can be used to granularly filter the publication's properties. In this example, we want `key: band`'s value to not equal to `Ramstein` AND `key: price`'s value has to be lower than `500`. Since this query returns true, we will get a match between the previous publication and this subscription.
* b. Use the wrapper MatchMore and call the method createSubscriptionForMainDevice() to send the creation request to Matchmore service.
Don't forget we need to handle two cases, in case of success we retrieve the created object, and in case of failure, we retrieve an error.

In case of success, Matchmore returns a new `Subscription` object with all the relevant details:
```JSON
{
  "id": "376880ff-52af-4424-8cc7-6033f01ec077",
  "createdAt": 1524232785653,
  "deviceId": "cc8ec61f-913c-4e81-b959-01153f8eb46d",
  "topic": "test",
  "range": 1,
  "duration": 60,
  "selector": "band <> 'Ramstein' AND price < 500",
  "pushers": [
      "localhost"
  ]
}
```
Once you create the subscription, store the id value in your own database for later reference (presumably with the user's deviceId).

#### STEP 4: When subscribers and publishers are within range of each other, there is a `Match`
A match warns the subscribers of the presence of nearby publishers.
In other words, `Device` that has `Subscription` attached to, are notified when `Match` occurs. It is not the case for `Device` with `Publication` attached to.

In order to receive a match, you need to create an instance which is conform to `MatchDelegate` protocol. All instances conform to `MatchDelegate` protocol and that is a delegate of `MatchMonitor`, are notified.

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
           // a. Get most recent Match
           let mostRecentMatch = matches.max(by: {
               $0.createdAt! < $1.createdAt!
           })

           // b. Unwrap properties
           guard let properties = mostRecentMatch?.publication?.properties else {
               print("No properties.")
               return
           }
           guard let band = properties["band"] as? String else {return}
           guard let tags = properties["tags"] as? Set else {return}
           guard let price = properties["price"] as? Int else {return}
           guard let currency = properties["currency"] as? String else {return}
           guard let free_parking = properties["free_parking"] as? Bool else {return}

           // c. Do something
       }
       // d. Add a new match delegate to Matchmore
       MatchMore.matchDelegates += self
   }
```

You need to create the delegate function for the onMatch listener. Every onMatchClosure are triggered when there is a new match.
The returned callback is composed of an array of hundred last matches, plus the concerned device. Then you decide what happens with the matches.
* a. Filter the Set of hundred returned last matches and get the most recent match.
* b. Safely unwrap the value you'll use.
* c. Use the match information as you need
```JSON
{
  "id": "c67dha4f-ce4a-4249-a1fa-33235732ce13",
  "createdAt": 1524232995653,
  "publication": {
    "id": "c62b5a4f-ce4a-4249-a1fa-33235732ce13",
    "createdAt": 1524212756568,
    "deviceId": "cc8ec61f-913c-4e81-b959-01153f8eb46d",
    "topic": "test",
    "range": 100,
    "duration": 60,
    "properties": {
      "band": "Metallica",
      "tags": [
        "rock",
        "metal"
      ],
      "price": 400,
      "currency": "CHF",
      "free_parking": true
    }
  },
  "subscription": {
    "id": "376880ff-52af-4424-8cc7-6033f01ec077",
    "createdAt": 1524232785653,
    "deviceId": "cc8ec61f-913c-4e81-b959-01153f8eb46d",
    "topic": "test",
    "range": 1,
    "duration": 60,
    "selector": "band <> 'Ramstein' AND price < 500",
    "pushers": [
        "localhost"
    ]
  }
}
```
* d. Earlier, we have made ViewController conform to MatchDelegate protocol. In order to be informed of every match, you'll need to add ViewController to Matchmore wrapper match delegates list.
Every time Matchmore SDK is informed of a new delegate, it will automatically include this new delegate of match occurences notification.

{: #modeling}
### Modeling
### Location and Device
TODO text qui parles de la liaison entre location et device
### Device
A device might be either virtual like a pin device or physical like a mobile device.
Devices may issue publications and subscriptions at any time; it may also cancel publications and subscriptions issued previously. Publications and subscriptions do have a definable, finite duration, after which they are deleted from the Matchmore cloud service and don’t participate anymore in the matching process.

### Location
A location consists of a geographical position of a Device. A conventionally known *update location* is a `Location` object sent to Matchmore cloud service to inform of changes upon position for a device.

* createdAt
integer <int64>
The timestamp of the location creation in seconds since Jan 01 1970 (UTC).

* latitude
number <double> Required
The latitude of the device in degrees, for instance '46.5333' (Lausanne, Switzerland).

* longitude
number <double> Required
The longitude of the device in degrees, for instance '6.6667' (Lausanne, Switzerland).

* altitude
number <double> Required
The altitude of the device in meters, for instance '495.0' (Lausanne, Switzerland).

* horizontalAccuracy
number <double>
The horizontal accuracy of the location, measured on a scale from '0.0' to '1.0', '1.0' being the most accurate. If this value is not specified then the default value of '1.0' is used.

* verticalAccuracy
number <double>
The vertical accuracy of the location, measured on a scale from '0.0' to '1.0', '1.0' being the most accurate. If this value is not specified then the default value of '1.0' is used.
#### Mobile
A mobile device is one that potentially moves together with its user and therefore has a geographical location associated with it. A mobile device is typically a location-aware smartphone, which knows its location thanks to GPS or to some other means like cell tower triangulation, etc.

* id
string
The id (UUID) of the device.

* createdAt
integer <int64>
The timestamp of the device's creation in seconds since Jan 01 1970 (UTC).

* updatedAt
integer <int64>
The timestamp of the device's creation in seconds since Jan 01 1970 (UTC).

* name
string
The name of the device.

* platform
string Required
The platform of the device, this can be any string representing the platform type, for instance 'iOS'.

* deviceToken
string Required
The deviceToken is the device push notification token given to this device by the OS, either iOS or Android for identifying the device with push notification services.

* location
Location Required

#### Pin
A pin device has an ultimate geographical location associated with it but is not represented by any object in the physical world; usually it’s location doesn’t change frequently if at all.

* id
string
The id (UUID) of the device.

* createdAt
integer <int64>
The timestamp of the device's creation in seconds since Jan 01 1970 (UTC).

* updatedAt
integer <int64>
The timestamp of the device's creation in seconds since Jan 01 1970 (UTC).

* name
string
The name of the device.

* location
Location Required
##### Using Pin Device

#### iBeacon
Beacons are high-tech tools that repeatedly broadcast a single signal under the form of advertising packet. Other devices interact with beacons through bluetooth and receive an advertising packet which consist of different letters and numbers. With the information received through the packet, devices like smartphones know how close they are to a specific beacon. The main purpose behind beacons is to improve indoor location. When developers know how close they are to this specific location, thanks to beacons, they can do something useful with this information.

* name
string
The name of the device.

* proximityUUID
string Required
The UUID of the beacon, the purpose is to distinguish iBeacons in your network, from all other beacons in networks outside your control.

* major
integer <int32> [ 0 .. 65535 ] Required
Major values are intended to identify and distinguish a group.

* minor
integer <int32> [ 0 .. 65535 ] Required
Minor values are intended to identify and distinguish an individual.
##### Beacons standard
The iBeacon is the standard defined by Apple. There is also other beacons standard like Eddystone by Google or AltBeacons by Kontakt.io. Basically, the content of the advertising packet could vary slightly from one standard to another, but the communication protocol (bluetooth) remains the same. As a consequence, most beacons on the market support at least the iBeacon standard and the Eddystone standard.

{: #ios}
### iOS
#### Versioning

SDK is written using Swift 4.

MatchmoreSDK requires iOS 9+.

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

#### Set up Matchmore Cloud Credentials
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
#### Apple Push Notification service
We use Apple Push Notification service (APNs). It allows app developers to propagate information via notifications from servers to iOS, tvOS and macOS devices.

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

You can now start sending notifications to your users.
#### Web Socket
When it comes to deliver matches, Matchmore SDK uses Web socket as well to provide alternative solutions to APNs.
By initializing this, you inform Matchmore Cloud service that you want to be notified via web socket.

```swift
    // MARK: - Socket
    func openSocketForMatches()
    func closeSocketForMatches()
```
#### Polling
Use these functions to start or stop polling matches from Matchmore Cloud.

```swift
    // MARK: - Polling
    func startPollingMatches(pollingTimeInterval: TimeInterval)
    func stopPollingMatches()
```
{: #android}
### Android
#### Standard Integration
#### Custom Integration
#### Google Firebase Cloud Notification
#### Web Socket
When it comes to deliver matches, ALPS uses Web socket as well to provide alternative solutions to APNs.
By initializing this, you inform Matchmore Cloud service that you want to be notified via web socket.
#### Polling
Use these functions to start or stop polling matches from Matchmore Cloud.
{: #unity}
### Unity 3D
#### Standard Integration
#### Custom Integration
{: #javascript}
### JavaScript
#### Standard Integration
#### Custom Integration
