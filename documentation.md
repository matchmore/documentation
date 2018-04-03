# Home
## Matchmore
Documentation

As the fast-growing world of mobile connected objects computing era is becoming a reality, Matchmore aims at becoming the leading cloud- based software platform supporting the creation of highly dynamic proximity-based applications. At the heart of our vision is the notion of geomatching, which builds on the location-based publish/subscribe communication model.

### Description

Our goal is to provide tools that dramatically simplify and accelerate the development, testing and deployment of rich application scenarios based on **multiple moving and connected objects**, making such applications the new low hanging fruits of the mobile app industry.

Get familiar with the Matchmore products and explore their features:

// links to our products, opens quickstart of || ALPS || PHOMO

### Understanding Matchmore
Matchmore helps you model any geolocated or proximity based applications, create any type of interactions for IoT by taking advantage of advanced location based publish/subscribe (ALPS) model - from smartphones to any sensors and everything in between.

#### In depth Publish/subscribe
The purpose of Publish/Subscribe model is to create communication between several computers, applications and users. Entities interested in a specific topic, subscribes to the topic and whenever there is a message on this topic, they will receive it. Content based filtering is an additional layer, namely selector, allowing subscribers to filter incoming messages based on the message’s content. On the other hand, publishers/senders are able to send messages without any knowledge of the future receivers, ensuring anonymity. Concerning content based filtering, publishers are able to customized their messages with additional layer, namely properties. In example, a publication and a subscription were issued on the same topic and have compatible properties and the evaluation of the selector against those properties returns true value, so there is a content correspondence which leads to a message delivery.

OR

The publish/subscribe model is a message-oriented paradigm in which, publishers can publish anonymously on a topic and subscribers can receive messages on that topic, additional layer, namely selector, allows subscribers to filter incoming messages based on the message’s content.

#### Advanced Location based Publish/Subscribe
ALPS is designed to simplify software developer’s life. ALPS is an extension of the Publish-Subscribe model adding user context while remaining message-oriented. By adding ranges to each publication/subscription, ALPS changes publish-subscribe paradigm into a context-aware paradigm. Delivery of message occurs based on the location of the device on which publication or subscription is attached. The crossing of the range and corresponding topic and content lead to the delivery of the message, in fact a match.

To facilitate the mobile application development, ALPS is provided in enhanced Software Development Kits (SDK).

So what exactly can you do with Matchmore ?!

### Geomatching (Matching Moving Content)
ALPS enables you to notify user whenever there’s someone they are interested of close by. Create proximity detection easily on web or mobile by using the advanced location based publish/subscribe paradigm.

### Create Smart Geofences
Geofences are used to define special areas in the real world, that you are particularly interested in.
With smart geofences you can go further in the filtering process. It can be used for registering how many app users interested in a specific topic enter a specific area (analytics uses), or for triggering content inside the mobile or externally depending on user's attached informations indoor (beacons) and outdoor (GPS).

### Smart activation with IoT
Activate a connected device when you approach it (a camera by example).

### Geochat
Broadcast anonymous message to people around you.

### Smart checkpoints
Create virtual gates or points to control passage of users. A trail is validated if the user has passed by every points related to the trail.

### And many other uses that we didn't think of yet .. !
We have a sample app where we grouped lot of interesting uses, here is a link to the Github.
ALPS paradigm is applicable to many domains and eases the modelisation of any geolocated applications.

### Some links to help (Examples of links : contact us, examples, tutorials)
### Redirection (What next, OR recently added)
### Questions ?
We're always happy to help with code or other questions you might have! Search our documentation, contact support, or connect with our sales team.

**The reader should understand Matchmore briefly and should be able to find his next step.**

# ALPS

## Quickstart
// I still think that quickstart should just go directly to 1. create a device 2. create pub 3. create sub 4. get match.
// All the info p list, cocoapods, configuration things should go somewhere else but where ?!

Quickly create proximity detection for any type of devices.

1. Create a device to **feature proximity detection**
2. Create a publication to **broadcast the presence of a device**
3. Create a subscription to **start discovering near devices**
4. When matches occur subscribers are notified of the presence of the publisher

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
A publication is similar to a Publish-Subscribe model publication extended with the notion of a geographical zone. The zone is defined as circle with a center at the given location and a range around that location. Publications and subscriptions which are associated with a mobile device, e.g. user’s mobile phone, potentially follow the movements of the user carrying the device and therefore change their associated location.

## Subscription
A subscription is similar to a Publish- Subscribe subscription extended with the notion of geographical zone. The zone is defined as circle with a center at the given location and a range around that location. Publications and subscriptions which are associated with a mobile device, e.g. user’s mobile phone, potentially follow the movements of the user carrying the device and therefore change their associated location.

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
