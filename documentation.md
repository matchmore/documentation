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
Matchmore SDK can be used in different ways, but the most common way is to have a main device. Usually, this main device is the device on which the app is running.

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

a. Use the wrapper MatchMore and call the method startUsingMainDevice() to start the registration of the app running device in Matchmore service.
b. MatchMore SDK's methods are generally having an asynchronous callback. To handle that, you need to declare closure. `result` is an enum, you need to unwrap it. When it is a success, the callback returns the object created in Matchmore's side, else when it is a failure, the callback is an error containing a message that describes what failed.
c. Start polling the matches.
d. Start informing Matchmore service that the user's main device is moving.

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
a. Create a publication and fulfill the required parameters.
* The topic is our first filter. In order to match, both a publication and a subscription need to be created on the same unique topic.
Here, the topic is *test*. Every publication that is set on topic *test* will concern only the *test* features. Use topic to manage different channels of matches.
* You can select the size of the zone around any publications/subscriptions, it is defined as a circle with a center at the given location and a range around that location. The range is defined in meters.
* You can also set the duration of your publications. The duration is defined in seconds.
* Now, you can set some properties to transmit either supplement information or to allow finer filtering for the subscribers of your topic.
The following data types are allowed in `Properties`: `String``Int``Set``Boolean`.
b. Use the wrapper MatchMore and call the method createPublicationForMainDevice() to send the creation request to Matchmore service.
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
When matches occur, all classes conform to `MatchDelegate` protocol is notified.

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

a. Safely unwrap the value you'll use.
b. Use the match information as you need
c. Earlier, we have made ViewController conform to MatchDelegate protocol. In order to be informed of every match, you'll need to add ViewController to Matchmore wrapper match delegates list.
## IOS (Configuration(Info.plist, CocoaPods, API-Key, MainDevice, Pub, Sub, Match, Start updating location))
### Standard Integration
We will use CocoaPods. In order to install this package manager you'll need to execute this command in the terminal:
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

### Custom Integration
### Apple Push Notification service
We use Apple Push Notification service (APNs) which is the main remote notification service provided by Apple. APNs allows app developers to propagate information via notifications from servers to iOS, tvOS and macOS devices.
### Web Socket
When it comes to deliver matches, ALPS uses Web socket as well to provide alternative solutions to APNs.
### Polling

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

### Beacon
Beacons are high-tech tools that repeatedly broadcast a single signal under the form of advertising packet. Other devices interact with beacons through bluetooth and receive an advertising packet which consist of different letters and numbers. With the information received through the packet, devices like smartphones know how close they are to a specific beacon. The main purpose behind beacons is to improve indoor location. When developers know how close they are to this specific location, thanks to beacons, they can do something useful with this information.
#### Beacons standard
The iBeacon is the standard defined by Apple. There is also other beacons standard like Eddystone by Google or AltBeacons by Kontakt.io. Basically, the content of the advertising packet could vary slightly from one standard to another, but the communication protocol (bluetooth) remains the same. As a consequence, most beacons on the market support at least the iBeacon standard and the Eddystone standard.

## Publication
A publication is similar to a Publish-Subscribe model publication extended with the notion of a geographical zone. The zone is defined as circle with a center at the given location and a range around that location. Publications which are associated with a mobile device, e.g. user’s mobile phone, potentially follow the movements of the user carrying the device and therefore change their associated location.

## Subscription
A subscription is similar to a Publish- Subscribe subscription extended with the notion of geographical zone. The zone is defined as circle with a center at the given location and a range around that location. Subscriptions which are associated with a mobile device, e.g. user’s mobile phone, potentially follow the movements of the user carrying the device and therefore change their associated location.

The following data types are allowed in Properties:  String, Int, Set, Boolean.

SQL Comparison Operators
=    Equal to    
/>    Greater than    
<    Less than    
/>=    Greater than or equal to    
<=    Less than or equal to    
<>    Not equal to

SQL Logical Operators
ALL    TRUE if all of the subquery values meet the condition    
AND    TRUE if all the conditions separated by AND is TRUE    
ANY    TRUE if any of the subquery values meet the condition    
BETWEEN    TRUE if the operand is within the range of comparisons    
IN    TRUE if the operand is equal to one of a list of expressions    
LIKE    TRUE if the operand matches a pattern    
NOT    Displays a record if the condition(s) is NOT TRUE    
OR    TRUE if any of the conditions separated by OR is TRUE    

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
