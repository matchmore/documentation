---
title: Proximity Detection
sections:
  - Quickstart
  - How it works
---

{: #quickstart}
### Quickstart
Quickly create proximity detection for any types of device.

Follow these four steps to create proximity detection:
1. Create a device to **feature proximity detection**
2. Create a publication to **broadcast the presence of a device**
3. Create a subscription to **start discovering near devices**
4. **Handle Match**, when subscribers and publishers are within range of each other

The devices are core models for proximity detection. A `Device` defines an object that is either virtual or physical which you can find useful when it is proximity detected. A device without publication or subscription is not detectable nor detecting, you need to attach `Publication` or `Subscription` on a device in order to start either **broadcast presence** or **discover nearby devices**.

`Match` is the event object to notify a proximity detection based on the encounter of two different devices.

#### STEP 1: Create a device to **feature proximity detection**
Matchmore relies on different sensors to provide the proximity detection; such as GPS, Bluetooth, and more. Each device is unique and offers distinct possibilities:
* `Mobile Device` is based on a constant `Location` update, thanks to GPS,
* `Beacon Device` is made for indoor and outdoor location services. It relies on Bluetooth technology,
* `Pin Device` is non-physical and at a fixed geographical location.
* More types of device will be added in the future.

Matchmore SDK can be used in different ways, but the most common way is to have a main device. Usually, this main device is the device on which the app is running, typically it represents your actual users; Smartphones (mobile app) or Browser (web app).

The following code demonstrates how to create the main device:
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
* b. MatchMore SDK's methods are generally having an asynchronous callback. To handle that, you need to declare a closure. Callback result is an enum, you need to unwrap it. When it is a success, the callback returns the object created in Matchmore's side, else when it is a failure, the callback is an error containing a message that describes what failed.

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
When a `Device` has a `Publication` attached to, it is as if it broadcasts its presence. You can see a publication as a message, you might store metadata, such as a phone number, a name, or any relevant information, on the publication object. Publications can contain customized information destined for future `Match` receivers. The metadata are grouped in the `properties` parameter.
You can attach as many publications and subscriptions as you want to a device. It means a device can be both publisher and subscriber at the same time. You may need to define just one publication/subscription or several hundred, depending on how many services you offer.
**Please be aware that publishers** are NOT notified of matches occurrence, if you want your publishers to be notified, he must be subscribing as well.

This code creates a publication via the SDK and this publication will be directly attached to the main device:
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
The following data types are allowed in `Properties` parameters: String, Int, Set and Boolean.
* b. Use the MatchMore SDK and call the method createPublicationForMainDevice() to send the creation request to Matchmore service.
N.B.: Don't forget we need to handle two cases, in case of success we retrieve the created object, and in case of failure, we retrieve an error.
In case of success, Matchmore returns a new publication object with all the relevant details:
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

#### STEP 4: **Handle Match**, when subscribers and publishers are within range of each other
A match warns the subscribers of the presence of nearby publishers.
In other words, device that has subscription attached to, are notified when `Match` occurs. It is not the case for device with only publication attached to.

In order to receive a match, you need to create an instance which is conform to `MatchDelegate` protocol. All instances conform to MatchDelegate protocol and that is a delegate of `MatchMonitor`, are notified.

Define an object that's conform to MatchDelegate protocol implementing OnMatchClosure.
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


{: #how-it-works}
### How it works
We run through all you need to know about Matchmore's service.
Fundamental objects and their interaction are described in details, so you can handle Matchmore by your own.

Matchmore service organizes the proximity detection with devices, publications and subscriptions. Devices are the central point of proximity detection; clients create devices to publish and subscribe to their interests, and every proximity detection, namely `match` is broadcast by Matchmore to the concerned subscribers.


### Object id
All objects in Matchmore are generated with a unique identifier, the id.

Besides id parameter, each type of object have their individual characteristics.

##### Objects states
Any objects in Matchmore can exist in any of the following states:

* creating

A creation has been initiated by sending a request to Matchmore. This is a transient state; it will be followed either by a transition to created, or failed

* Created

Creation has succeeded. In the created state an object is considered active in Matchmore. It has a unique id generated by Matchmore and a time of creation.

* deleting

A deletion has been initiated on the created objects by sending a request to Matchmore. This is a transient state; it will be followed either by a transition to deleted or failed

* Deleted

The Objects, having previously been created, has been deleted by the user

<div class="callout-block callout-danger"><div class="icon-holder">*&nbsp;*{: .fa .fa-exclamation-triangle}
</div><div class="content">
{: .callout-title}
#### failed creation or deletion

Matchmore always try to inform you with error messages if any creation or deletion has failed. This state is entered if an error has been received from the Matchmore service (such as an attempt to create without the necessary access rights).

</div></div>


### Location and Device
The concept of `Location` and `Device` is very important to understand how to use Matchmore.
In Matchmore, users interact with each other via devices. Each device can be detected by any other devices. Generally each device is at one location. This location can be fixed or mobile (changing over time) depending on the type of devices.


{: #location}
### Location
A location consists of a geographical position of a Device. A conventionally known *update location* is a `Location` object sent to Matchmore cloud service to inform of changes upon position for a device.


{: #device}
### Device
A device might be either virtual like a pin device or physical like a mobile or beacon device.
In order to publish or subscribe to, you must first obtain a device instance and then attach pub/sub to that device. In most instances, as a convenience, it is unnecessary to explicitly attach to a device as it will implicitly attached to your main device when performing any publishing or subscribing.
Devices may issue publications and subscriptions at any time; it may also cancel publications and subscriptions issued previously.

In the following examples, you will learn how to obtain each types of device instance.

#### Mobile
A mobile device is one that potentially moves together with its user and therefore has a geographical location associated with it. A mobile device is typically a location-aware smartphone, which knows its location thanks to GPS or to some other means like cell tower triangulation, etc.


##### Example obtain a Mobile device
When starting your application, you want to make sure a Mobile device is corresponding to your smartphone running app in Matchmore's end.
By calling method `startUsingMainDevice()`, you can either create a new mobile device or reuse one that you are aware that it exists in Matchmore cloud service.

1. **Create a new mobile device**

To create a new mobile device, call function `startUsingMainDevice()` with empty parameters.
If you are using Matchmore SDK, then it automatically creates a mobile device in the Matchmore cloud for you.

```swift
MatchMore.startUsingMainDevice { result in
    guard case .success(let mainDevice) = result else { print("🌋 \(String(describing: result.errorMessage)) 🌋"); return }
    print("🏔 Using device: 🏔\n\(mainDevice.encodeToJSON())")
}
```

2. **Reuse a mobile device**
First, you need to be able to pass the ID of an existing mobile device. Presumably, it should be locally stored on the smartphone memory.

```swift
var myMainDevice : MobileDevice?
MatchMore.mobileDevices.find(byId: "ID stored in your backend", completion: {result in
    if (result != nil){
        myMainDevice = result
        MatchMore.startUsingMainDevice(device: myMainDevice, shouldStartMonitoring: true, completion: {result in
            switch result{
            case .success(let mobile):
                print("🏔 Main Device successfully retrieved 🏔\n\(mobile.encodeToJSON())")
                myMainDevice = mobile
            case .failure(let error):
                print("🌋 \(String(describing: error?.message)) 🌋")
            }
        })
    } else {
        print("The id used to retrieve a main device does not correspond to any known mobile devices.")
    }
})
```


#### Pin
A pin device has an ultimate geographical location associated with it but is not represented by any object in the physical world; usually it’s location doesn’t change frequently if at all.

##### Example obtain a pin device


```swift
let locationTest = Location.init(latitude: 46.490513, longitude: 6.430700)
let myPinDevice = PinDevice.init(name: "pin test", location: locationTest)
MatchMore.createPinDevice(pinDevice: myPinDevice, completion: { (result) in
    switch result{
    case .success(let pin):
        print("🏔 Pin device was created 🏔\n\(pin.encodeToJSON())")
    case .failure(let error):
        print("🌋 \(String(describing: error?.message)) 🌋")
    }
})
```

#### iBeacon
Beacons are high-tech tools that repeatedly broadcast a single signal under the form of advertising packet. Other devices interact with beacons through bluetooth and receive an advertising packet which consist of different letters and numbers. With the information received through the packet, devices like smartphones know how close they are to a specific beacon. The main purpose behind beacons is to improve indoor location. When developers know how close they are to this specific location, thanks to beacons, they can do something useful with this information. Proximity detection with beacons is done via Bluetooth technology, as a result, the beacons are not geographically located with geographic coordinates.

##### Example obtain a beacon device
Beacon device works slightly different from Mobile and Pin.
It requires a pre-registration of all beacons in our portal.

1. Go on Matchmore portal and start registering the beacons that you are going to use.
We need you to provide us their UUID, major and minor parameters.

2. Stay on Matchmore portal, and go to your applications view.
Assign the beacons to the applications that will use them.
When it is done, all registered beacons are accessible with right credentials via REST API or SDKs.

3. Get your registered beacons in your app and start publishing to or subscribing to.
```swift
MatchMore.knownBeacons.findAll(completion: { beacons in
    // store the beacon that you are looking for
})
```

<div class="callout-block callout-info"><div class="icon-holder">*&nbsp;*{: .fa .fa-info-circle}
</div><div class="content">
{: .callout-title}
#### Beacons standard

The iBeacon is the standard defined by Apple. There is also other beacons standard like Eddystone by Google or AltBeacons by Kontakt.io. Basically, the content of the advertising packet could vary slightly from one standard to another, but the communication protocol (bluetooth) remains the same. As a consequence, most beacons on the market support at least the iBeacon standard and the Eddystone standard.
</div></div>


#### Publication & Subscription
When devices are created, you can start to attach publications and subscriptions. A publication is a message extended with the notion of a geographical zone. The zone is defined as a circle with a center at the given location of the attached device and a range around that location. Publications and subscriptions do have a definable, finite duration, after which they are deleted from the Matchmore cloud service and don’t participate anymore in the matching process.

###### Publication

<div class="table-responsive">

{: .table .table-striped}
| Parameters | Description
|-
| id | The id (UUID) of the publication
| createdAt | The timestamp of the publication creation in seconds since Jan 01 1970 (UTC)
| deviceId | The id (UUID) of the device to attach a publication to
| topic | The topic of the publication. This will act as a first match filter. For a subscription to be able to match a publication they must have the exact same topic
| range | The range of the publication in meters. This is the range around the device holding the publication in which matches with subscriptions can be triggered
| duration | The duration of the publication in seconds. If set to '0' it will be instant at the time of publication. Negative values are not allowed
| properties | The dictionary of key, value pairs. Allowed values are number, boolean, string and array of aforementioned types

</div>

###### Subscription

<div class="table-responsive">

{: .table .table-striped}
| Parameters | Description
|-
| id | The id (UUID) of the publication
| createdAt | The timestamp of the publication creation in seconds since Jan 01 1970 (UTC)
| deviceId | The id (UUID) of the device to attach a publication to
| topic | The topic of the publication. This will act as a first match filter. For a subscription to be able to match a publication they must have the exact same topic
| selector | This is an expression to filter the publications. For instance 'job='developer'' will allow matching only with publications containing a 'job' key with a value of 'developer'
| range | The range of the publication in meters. This is the range around the device holding the publication in which matches with subscriptions can be triggered
| duration | The duration of the publication in seconds. If set to '0' it will be instant at the time of publication. Negative values are not allowed
| pushers | When match will occurs, they will be notified on these provided URI(s) address(es) in the pushers array

</div>

##### Topic
The Matchmore service organizes the traffic within any application into named topics. Clients connected to Matchmore can either be publishers (they announce their presence via messages), subscribers (they look for discovering nearby devices), or both. Publications (messages) are always published over a named topic. Topics are used to filter matches, so whilst billions of devices may cross each other by Matchmore, subscribers will only receive the matches on the topics they subscribe to.

Topics are uniquely identified by a string specified when publishing to or subscribing to a topic. Publishers and subscribers are completely decoupled:
* a publisher can publish a message without any subscribers on the topic, but when the publication's zone and a subscription's zone overlap there will be a match;
* subscribers can listen on topics that don’t yet have publishers, publishers can arrive later and if both zones overlap there is a match;
* arbitrarily many subscribers can match with a single publication published on a topic;
* The other way is also possible, many publishers can match with a single subscribers subscribing on a topic.
In other words, Matchmore topics support one-to-many, many-to-one, and many-to-many.


##### Properties & selector
Inside of publications messages and subscriptions, Matchmore allows accurate filtering. Thanks to parameters `properties` and `selector`.

* Publishers :

publications are entirely customizable, via parameter `properties`.
Properties is a Map, you can set keys and values pairs to transmit supplement metadata to allow finer filtering for the subscribers of your topic.
The following data types are allowed in Properties: String, Int, Set and Boolean.

* Subscribers :

`selector` parameter let subscribers filter incoming matches based on the publication's content (properties).

Here is a list of operators you can use to filter incoming matches.

<div class="table-responsive">

{: .table .table-striped}
| Supported SQL operators | Description
|-
| Comparison Operators |
| = | Equal to
| > | Greater than
| < | Less than
| >= | Greater than or equal to
| <= | Less than or equal to
| <> | Not equal to
| Logical Operators |
|-
| ALL | TRUE if all of the subquery values meet the condition
| AND | TRUE if all the conditions separated by AND is TRUE
| ANY | TRUE if any of the subquery values meet the condition
| BETWEEN | TRUE if the operand is within the range of comparisons
| IN | TRUE if the operand is equal to one of a list of expressions
| LIKE | TRUE if the operand matches a pattern
| NOT | Displays a record if the condition(s) is NOT TRUE
| OR | TRUE if any of the conditions separated by OR is TRUE

</div>

I.e., a publication and a subscription were issued on the same topic and have compatible properties and the evaluation of the selector against those properties returns true value, so there is a content correspondence which could lead to a match delivery (match occurs when both zones overlap).


##### Range
When publishing or subscribing, you can decide the range in which it is interesting to start being discovered or discovering.
The range is defined in meters.
Upper limit is 5000 meters and lower limit is 1 meter.


##### Duration
When publishing or subscribing, you decide how long it is useful for you to be discovered or discovering other devices.
The duration is defined in seconds.
There is no upper limit to duration and you can't set negative duration.


{: #pushers}
##### Subscription's Pushers
With pushers, you can choose **how and where** you want to be notified when matches occur.
You can stream the realtime matches within the Matchmore platform directly to another streaming, queueing service, devices or servers.
For example, you could persist each match of a user to your own database, start counting matches, trigger an event if a device has matched with a location that indicates that it has reached its destination, trigger execution of code on your servers. Pushers help solve many of the challenges associated with consuming matches data server-side or client-side.


##### Example
You are a company selling food and you want to count how many of your app users food hungry pass-by one of your store.
This data is useful to know when to open your store for example.

1. Create your store as a `Pin device`.
```swift
// Company's store location
let locationTest = Location.init(latitude: 46.490513, longitude: 6.430700)
var myPinDevice = PinDevice.init(name: "pin test", location: locationTest)
MatchMore.createPinDevice(pinDevice: myPinDevice, completion: { (result) in
    switch result{
    case .success(let pin):
        // Store this created Pin in the variable myPinDevice
        myPinDevice = pin
        // Start monitoring for matches
        MatchMore.startMonitoringMatches(forDevice: pin)
    case .failure(let error):
        print("🌋 \(String(describing: error?.message)) 🌋")
        // make sure your variable myPinDevice doesn't no contain a device that does not exist
        myPinDevice = nil
    }
})
```

2. You know that each of your app users have a publication on the topic "pass-by-store" attached to the main device.
For example, some information are known when the users login to your app. These information are then set in the publication.
```swift
MatchMore.startUsingMainDevice { result in
    let userProperties : [String: Any] = ["name":"Jon",
                                          "age": 24,
                                          "gender":"male",
                                          "starving": true]
    let publication = Publication.init(topic: "pass-by-store", range: 10, duration: 5000, properties: userProperties)
    MatchMore.createPublicationForMainDevice(publication: publication, completion: {result in
        switch result {
        case .success(let publication):
            print(publication.encodeToJSON())
        case .failure(let error):
            print("🌋 \(String(describing: error?.message)) 🌋")
        }
    })
}
```

3. Finally, you need to attach a subscription to the store (created pin device at point 1) on the topic "pass-by-store".
Remember, you are interested only for food hungry users, so you use selector to filter their interest.
Each day is 86400 second long so you set that duration. You are now ready to detect any of your app users passing-by your store.
```swift
let subscription = Subscription.init(topic: "pass-by-store", range: 5, duration: 86400, selector: "starving = true")
MatchMore.createSubscription(subscription: subscription, forDevice: myPinDevice, completion: {result in
    switch result{
    case .success(let sub):
        print(sub.encodeToJSON())
    case .failure(let error):
        print("🌋 \(String(describing: error?.message)) 🌋")
    }
})
```


#### Match
A match between a publication and a subscription occurs when both of the following two conditions hold:

1. There is a context match occurs when for instance the subscription zone overlaps with the publication zone or a proximity event with an iBeacon device within the defined range occurred.

2. There is a content match: the publication and the subscription match with respect to their counterparts namely, properties and selector, i.e., they were issued on the same topic and have compatible properties and the evaluation of the selector against those properties returns true value.

Whenever a match between a publication and a subscription occurs, the device which owns the subscription receives that match asynchronously via a push notification if there exists a registered push endpoint. A push endpoint is an URI which is able to consume the matches for a particular device and subscription. The push endpoint doesn't necessary point to a mobile device but is rather a very flexible mechanism to define where the matches should be delivered. Look for [pushers](#pushers) for more information.

Matches can also be retrieved by issuing a API call for a particular device.


##### Example
Any instances can handle matches for you.
You need to make the class of your handler conform to MatchDelegate protocol and implement onMatchClosure.

Here is an example of class.
*MatchHandler.swift*
```swift
class MatchHandler: MatchDelegate {
    var onMatch: OnMatchClosure?
    init(_ onMatch: @escaping OnMatchClosure) {
        self.onMatch = onMatch
    }
}
```

Create an instance of MatchHandler class and add it to match delegates.
```swift
let matchHandler = MatchHandler.init({ matches, device in
    print(matches.description)
    print(device)
})
MatchMore.matchDelegates.add(matchHandler)
```

When you have described the code that must be run when matches occur, just delegate your instances conform to MatchDelegate protocol to MatchDelegate. In this example, for every match there will be a message printed in the console describing all the received matches, and the related device.
