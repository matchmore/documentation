---
title: Geomatching
sections:
  - Quickstart
  - How it works
---

{: #quickstart}
### Quickstart
Quickly start geomatching for any type of device.

Follow these four steps to start geomatching:
1. Create a device to **feature geomatching**
2. Create a publication to **broadcast the presence of a device**
3. Create a subscription to **start discovering nearby devices**
4. **Handle the Match**, when subscribers and publishers are within range of each other

The devices are core elements of the domain model for geomatching. A `Device` defines an object that is either virtual or physical. A device without publication or subscription is not detectable nor detecting, so you need to attach `Publication` or `Subscription` on a device in order to start either to **broadcast presence** or to **discover nearby devices**.

A `Match` is the event object to notify a device of any geomatching. It is location and content based, in other words a `Match` event is triggered only if both devices are in the specified range and their Publication/Subscription content match.

#### STEP 1: Create a device to **feature geomatching**
To provide geomatching Matchmore relies on different hardware sensors such as GPS, Bluetooth, and others. Each device is unique and offers distinct possibilities:
* `Mobile Device` is providing `Location` updates, thanks to its GPS
* `Beacon Device` is made for indoor and outdoor location services. It relies on Bluetooth technology.
* `Pin Device` is a virtual (non-physical) device and placed at geographic coordinates.
* More types of device will be added in the future.

Matchmore SDK can be used in different ways, but the most common way is to have a main device. Usually, this main device is the device on which the app is running, typically it is operated by your actual user: Smartphone (mobile app) or Browser (web app).

The following code demonstrates how to create the main device:
```swift
// a. Create the main device
MatchMore.startUsingMainDevice { result in
    // b. Unwrap the result
    guard case .success(let mainDevice) = result else { print(result.errorMessage ?? ""); return }
    print("Using device: \n\(mainDevice.encodeToJSON())")

    // c. Start getting matches
    MatchMore.startPollingMatches()
    // d. Start location track in Matchmore
    MatchMore.startUpdatingLocation()
}
```

* a. Use the wrapper MatchMore and call the method startUsingMainDevice() to start the registration of the application running on your device in Matchmore service.
* b. MatchMore SDK's methods are generally using an asynchronous callback. To handle it, you need to declare a closure. Callback's result is an enum, so you need to unwrap it. When it is a success, the callback returns the `Device` object that has been created, when it is a failure, the callback provides you with an `Error` object containing a message to describe why it has failed.

In case of success, Matchmore returns a new device object with all of the relevant details:
```json
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

Once you have created a device, we advise you to store it's id value in your own database for later reference.
Devices have a universally unique identifier (UUID) that is generated automatically upon device creation.

When creating a device through the API, the response includes an UUID, which you can use to attach a `Publication` or `Subscription` to this `Device`. To attach a Publication/Subscription to an existing device provide it's UUID in your Matchmore requests. Pass the UUID of the desired device as the deviceId value of the Publication/Subscription parameter.
* c. When using our SDK, polling the `Match` in Matchmore cloud service is easy, just one line of code.
* d. Again, using our SDK will relieve you of many tasks. This single line of code starts informing Matchmore service of the user's device location.

#### STEP 2: Create a publication to **broadcast the presence of a device**
When a `Device` has a `Publication` (or a `Subscription`) attached to it, it will broadcast it's presence. One can think about a publication as a message, that might store json properties such as a phone number, a name, or any relevant information. Publications can contain customized information for future `Match` receivers. They are grouped in the `properties` parameter.
You can attach as many `Publication` and `Subscription` to a `Device` as you want. A device can be both publisher and subscriber at the same time. You may define one Publication/Subscription or several hundred, depending on scenarios.
**Publishers** are NOT notified of matches occurrence, if you want your publishers to be notified, they must be subscribing as well.

This code creates a publication via the SDK and the publication will be directly attached to the main device:
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
        print("Pub was created: \n\(publication.encodeToJSON())")
    case .failure(let error):
        print("\(String(describing: error?.message))")
    }
  })
}
```
* a. Create a publication and fill the required parameters.
- The topic is our first filter. In order to match, both a `Publication` and a `Subscription` need to be created on the same unique topic.
Here, the topic is *test*. We use the topic to filter different matches channels.
- You can select the range of the zone around any publications/subscriptions, it is defined as a circle with a center at the given location and a radial distance around that location. The range is defined in meters.
- You have to set the duration of your publications. The duration is defined in seconds.
- Now, you can set some properties to allow finer filtering for the subscribers of your topic.
The following data types are allowed in `Properties` parameters: String, Int, Set and Boolean.
* b. Use the MatchMore SDK and call the method createPublicationForMainDevice() to send a creation request to the Matchmore service.
Don't forget that we need to handle two cases: in case of success we retrieve the created object, and in case of failure, we retrieve an error.
In case of success, Matchmore returns a new `Publication` object with all the relevant details:
```json
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
Once you have created the publication, store the id value in your own database for later reference (presumably with user's deviceId).

#### STEP 3: Create a subscription to **start discovering nearby devices**
When a `Match` occurs, every **subscriber** receives a message based on their topic and content selection. With the selector parameter, you can filter publications based on the metadata contained within them to discover only the devices that might interest you.

Matchmore offers the ability to simultaneously subscribe a device to multiple topics. This provides additionnal channel to easily include your features in flexible ways. For example, if your service offers multiple features or services based on proximity detection, each could be identified by its own topic and the same device could be subscribed to more than one topic. Imagine an application that has two features: geo-chatting (Broadcast messages to nearby users) and geomatching (discover how many users are nearby).
You can create the features on two different topics for example, "geochat" for geochatting and "detection" for geomatching and to completely separate both features.

```swift
// a. Create a Subscription, set topic and selector.
let subscription = Subscription(topic: "test", range: 1, duration: 60, selector: "band <> 'Ramstein' AND price < 500")
// b. Create a Subscription for Main Device
MatchMore.createSubscriptionForMainDevice(subscription: subscription, completion: { result in
    switch result {
    case .success(let sub):
        print("Socket Sub was created \n\(sub.encodeToJSON())")
    case .failure(let error):
        print("\(String(describing: error?.message))")
    }
})
```

* a. Create a subscription to listen for matches on the `test` topic.
- Remember that the topic is the first level of filtering you can use.
- As for `Publications`, you can set the desired range and duration for `Subscriptions`.
- The selector can be used to granularly filter the publication's properties. In this example, we want `key: band`'s value to be different from `Ramstein` and `key: price`'s value to be lower than `500`. As soon as the two devices will be in range and since this query will be evaluated to **True**, we will get a match between the previous publication and this subscription.
* b. Use the wrapper MatchMore and call the method createSubscriptionForMainDevice() to send the creation request to Matchmore service.
Don't forget that we need to handle two cases: in case of success we retrieve the created object, and in case of failure, we retrieve an error.

In case of success, Matchmore returns a new `Subscription` object with all the relevant details:
```json
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
Once you create a `Subscription`, we advise you to store it's id value in your own database for later reference (presumably with the deviceId of the user).

#### STEP 4: **Handle Match**, when subscribers and publishers are within range of each other geographically and topic and contents are matching.
A match warns the subscribers of the presence of nearby publishers.
In other words, a device that has a subscription attached to it is notified when a `Match` occurs. It is not the case for devices with publications attached to them.

In order to receive a match, you need to create an instance which is conform to `MatchDelegate` protocol. All instances that conform to MatchDelegate protocol and that are delegate of `MatchMonitor`, are going to be notified.

Define an object that is conform to MatchDelegate protocol implementing OnMatchClosure.
```swift
class ExampleMatchHandler: MatchDelegate {
    var onMatch: OnMatchClosure?
    init(_ onMatch: @escaping OnMatchClosure) {
        self.onMatch = onMatch
    }
}
```

In this example we ensure that class ViewController is conform to the MatchDelegate protocol.
You also need to add var onMatch which is a closure.
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

You need to create a delegate function for the onMatch listener. Every onMatchClosure are triggered when there is a new match.
The returned callback is composed of an array containing the hundred last matches, plus the device. Then you decide what happens with the matches:
* a. Filter the Set of hundred returned last matches and get the most recent one.
* b. Safely unwrap the value you'll use.
* c. Use the match information that you need
```json
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
Every time Matchmore SDK is notified of a new delegate, it will automatically include this new delegate to the list of objects that are notified for matches.


{: #how-it-works}
### How it works
We will run through all what you need to know about Matchmore's service.

Fundamental objects and their interaction are described in details, so you can handle Matchmore on your own.

Matchmore's geomatching relies on devices, publications and subscriptions. Devices are the central element for geomatching; clients create devices to publish and subscribe, and every geomatching, namely `match` is broadcasted by Matchmore to the concerned subscribers.


### Object id
All objects in Matchmore are generated with a unique identifier, the id.

Besides the id parameter, each type of object have their individual characteristics.

##### Objects states
Any objects in Matchmore can exist in any of the following states:

* Creating

A creation has been initiated by sending a request to Matchmore. This is a transient state; it will be followed either by a transitiion to created or failed.

* Created

Creation has succeeded. At this point, an object is considered being active and alive in Matchmore. It has a unique id generated by Matchmore and a creation datetime property.

* Deleting

A deletion has been initiated on the object by sending a request to Matchmore. This is a transient state; it will be followed either by a transition to deleted or failed

* Deleted

The object has been deleted in Matchmore, it is therefore no longer active, nor alive.

* Failed

The action has failed, see below for details.

<div class="callout-block callout-danger"><div class="icon-holder">*&nbsp;*{: .fa .fa-exclamation-triangle}
</div><div class="content">
{: .callout-title}
#### failed creation or deletion

Matchmore tries to always inform you with error messages when creation or deletion has failed. This state is entered if an error has been received from the Matchmore service (i.e. an attempt to create without the necessary access rights).

</div></div>


### Location and Device
The concept of `Location` and `Device` is very important to understand how to use Matchmore.
In Matchmore, users interact with each other via devices. Each device can be detected by any other devices. Usually, a device is at one location. This location can be static or mobile (changing over time) depending on the type of devices.


{: #location}
### Location
A location consists of the geographical coordinates of a Device. An *update location* is a `Location` object sent to Matchmore cloud service to update a device position when it has moved.


{: #device}
### Device
A device might be either virtual like a pin device or physical like a mobile or a beacon device.
In order to publish or subscribe to it, you must first obtain a device instance and then attach your `Publication` or `Subscription` to that device. In most case it is unnecessary to explicitly attach to a specific device because the sdk will implicitly use your main device for that purpose.
Devices may issue publications and subscriptions at any time; they may also cancel publications and subscriptions issued previously.

In the following examples, you will learn how to obtain each types of device instances.

#### Mobile
A mobile device is one that potentially moves with its owner and therefore has a geographical location associated with it. A mobile device is typically a location-aware smartphone, which knows its location thanks to GPS or to some other means like cell tower triangulation, etc.


##### Example: How to obtain a Mobile device
When starting your application, you want to make sure that a Mobile device corresponding to your actual smartphone running your applications already exists in Matchmore's backend.

By always calling method `startUsingMainDevice()` when you start your Application, it will automatically either create a new mobile device or reuse the one previously created and stored locally by Matchmore's sdk.

1. **Use the current mobile device**

To use the current mobile device, call function `startUsingMainDevice()` with empty parameters.

```swift
MatchMore.startUsingMainDevice { result in
    guard case .success(let mainDevice) = result else { print("\(String(describing: result.errorMessage))"); return }
    print("Using device: \n\(mainDevice.encodeToJSON())")
}
```

2. **Use a particular device**

If you need to use a specific `Device` as your main device, you may pass any deviceId to `startUsingMainDevice()` as shown below.


```swift
var myMainDevice : MobileDevice?
MatchMore.mobileDevices.find(byId: "ID stored in your backend", completion: {result in
    if (result != nil){
        myMainDevice = result
        MatchMore.startUsingMainDevice(device: myMainDevice, shouldStartMonitoring: true, completion: {result in
            switch result{
            case .success(let mobile):
                print("Main Device successfully retrieved \n\(mobile.encodeToJSON())")
                myMainDevice = mobile
            case .failure(let error):
                print("\(String(describing: error?.message))")
            }
        })
    } else {
        print("The id used to retrieve a main device does not correspond to any known mobile devices.")
    }
})
```


#### Pin

A pin device has geographical coordinates associated with it but is not represented by any object in the physical world; usually its location doesnt change frequently if at all.

##### Example: How to obtain a Pin device

```swift
let locationTest = Location.init(latitude: 46.490513, longitude: 6.430700)
let myPinDevice = PinDevice.init(name: "pin test", location: locationTest)
MatchMore.createPinDevice(pinDevice: myPinDevice, completion: { (result) in
    switch result{
    case .success(let pin):
        print("Pin device was created \n\(pin.encodeToJSON())")
    case .failure(let error):
        print("\(String(describing: error?.message))")
    }
})
```

#### Beacon

Beacons are hardware devices that repeatedly broadcast a single signal under the form of an advertising packet. Other devices interact with them through bluetooth and receive their advertising packet which consist of different letters and numbers. With the received information, devices like smartphones know how close they are to a specific beacon. The main purpose of beacons is to handle indoor proximity detection. When developers know how close they are to a specific location, thanks to a nearby beacon, they can do something useful with this information. Geomatching with beacons is done via Bluetooth technology, as a result, the beacons are not geographically located with geographic coordinates. With Matchmore, you are able to attach information to your beacons, thanks to publication's properties.

##### Example: How to obtain a Beacon device

Beacon device works slightly differently from Mobile and Pin devices.
To operate, they require a pre-registration in our web portal. This pre-registration is mandatory because on most platforms it is only possible to detect known beacons.

1. Go on Matchmore portal and start registering the beacons that you are going to use.
To identify them, we need you to provide us their UUID, major and minor parameters.

2. Stay on Matchmore portal, and go to your applications view.
Assign the beacons to the applications that will use them.
When it is done, all registered beacons are accessible with proper credentials via REST API or SDKs.

3. Get your registered beacons in your app and start publishing to or subscribing to them.
```swift
MatchMore.knownBeacons.findAll(completion: { beacons in
    // store the beacon that you are looking for
})
```

<div class="callout-block callout-info"><div class="icon-holder">*&nbsp;*{: .fa .fa-info-circle}
</div><div class="content">
{: .callout-title}
#### Beacons standard

iBeacon is the standard defined by Apple. There is also other beacons standards like Eddystone by Google or AltBeacons by Kontakt.io. Basically, the content of the advertising packet could vary slightly from one standard to another, but the communication protocol (bluetooth) remains the same. As a consequence, the large majority of beacons on the market support at least the iBeacon standard and the Eddystone standard. IBeacon standard is fully supported by Matchmore.
</div></div>


#### Publication & Subscription
When devices are created, you can start to attach publications and subscriptions to them. A `Publication` is a message extended with the notion of a geographical zone. The zone is defined as a circle with a center at the given location of the attached device and a range around that location. Publications and subscriptions do have a definable, finite duration, after which they are deleted from the Matchmore cloud service and don't participate anymore in the matching process.

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
The Matchmore service organizes the traffic within any application into named topics. Clients connected to Matchmore can either be publishers (they announce their presence via messages), subscribers (they want to discover nearby information), or both. Publications (messages) are always published over a named topic. Topics are used to filter matches, so whilst billions of devices may cross each other, subscribers will only receive the matches on the topics they subscribe to.

Topics are uniquely identified by a string specified when publishing or subscribing to a topic. Publishers and subscribers are asynchronous:
* a publisher can publish a message without any subscribers on that particular topic, but later when a `Subscription` is created on that topic and when the ranges overlap there will be a match;
* subscribers can listen on topics that don't have publishers yet, similarly if a `Publication` is created later on and if both zones overlap there will be a match;
* arbitrarily many subscribers can match with a single publication published on a topic;
* the other way around is also possible, many publishers can match with a single subscriber on a topic.
In other words, Matchmore topics handle one-to-many, many-to-one, and many-to-many relationships.


##### Properties & selector
Inside of publications messages and subscriptions, Matchmore allows accurate filtering. Thanks to parameters `properties` and `selector`.

* Publishers :

Publications are customizable, using their `properties` parameter.

Properties are a Map Data Structure, where you can set keys and values pairs to allow finer filtering.
The following data types are allowed in Properties: String, Int, Set and Boolean.

* Subscribers :

`Selector` parameter allow the subscribers to filter incoming matches based on the publication's properties.

Here is a list of operators you can use to filter incoming matches:

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
You can stream matches directly to another queueing service, device or server.
For example, you could persist each match of a user to your own database, start counting matches, trigger an event if a device has matched with a location that indicates that it has reached its destination, trigger execution of code on your server and so on. Pushers help to solve many of the challenges associated with consuming matches data server-side or client-side.


##### Example of use for publications and subscriptions
Imagine a French tourist application in the city of Paris. The tourism office is interested to know how many of their application's users pass-by the Eiffel tower.
This data may, for instance, be useful to anticipate rush hours.

1. Create a `Pin device` to represent the Eiffel tower.
```swift
// Monument's location
let eiffelTowerLocation = Location.init(latitude: 48.858093, longitude: 2.294694)
var myEiffelPin = PinDevice.init(name: "Eiffel tower", location: eiffelTowerLocation )
MatchMore.createPinDevice(pinDevice: myEiffelPin, completion: { (result) in
    switch result{
    case .success(let pin):
        // Store this created Pin in the variable myEiffelPin
        myEiffelPin = pin
        // Start monitoring for matches
        MatchMore.startMonitoringMatches(forDevice: pin)
    case .failure(let error):
        print("\(String(describing: error?.message))")
        // make sure your variable myPinDevice doesn't no contain a device that does not exist
        myEiffelPin = nil
    }
})
```

2. You know that each of your application's user has a publication on the topic "eiffel-tower" attached to their main device.
Some useful information are known when the users login to your app, let's save this information in the publication properties for later use.
```swift
MatchMore.startUsingMainDevice { result in
    let userProperties : [String: Any] = ["name":"Jon",
                                          "age": 24,
                                          "gender":"male",
                                          "isTourist": true]
    let publication = Publication.init(topic: "eiffel-tower", range: 10, duration: 5000, properties: userProperties)
    MatchMore.createPublicationForMainDevice(publication: publication, completion: {result in
        switch result {
        case .success(let publication):
            print(publication.encodeToJSON())
        case .failure(let error):
            print("\(String(describing: error?.message))")
        }
    })
}
```

3. Finally, you need to attach a subscription to the Eiffel tower itself created above as a pin Device.
Each day is 86400 second long so you set that duration to have a daily `Subscription`. You are now ready to detect any of your app users passing-by the Eiffel-tower.
```swift
let subscription = Subscription.init(topic: "eiffel-tower", range: 50, duration: 86400, selector: "isTourist = true")
MatchMore.createSubscription(subscription: subscription, forDevice: myEiffelPin, completion: {result in
    switch result{
    case .success(let sub):
        print(sub.encodeToJSON())
    case .failure(let error):
        print("\(String(describing: error?.message))")
    }
})
```


#### Match
A match between a `Publication` and a `Subscription` occurs when both of the following two conditions hold:

1. There is a *context match**: when the subscription zone overlaps with the publication zone. It also works with beacons based on the proximity distance detection.

2. There is a **content match**: the publication and the subscription match with respect to their properties and selector, i.e., they were issued on the same topic and the evaluation of the selector against the properties returns a **True** evaluation.

Whenever a match between a `Publication` and a `Subscription` occurs, the device owning the subscription receives that match asynchronously via a push notification, if a push endpoint has been defined. A push endpoint is an URI which is able to consume matches. The push endpoint doesn't necessary point to a mobile device but is rather a very flexible mechanism to define where the matches should be delivered. More than one push endpoint can be active on the same `Subscription` which means that you can push your matches to different locations at the same time. Look for [pushers](#pushers) for more information.

Matches can also be retrieved by issuing a API call for a particular device.


##### Example: How to handle a match
You need to make the class of your handler conform to MatchDelegate protocol and implement onMatchClosure.

Here is an example of class:
*MatchHandler.swift*
```swift
class MatchHandler: MatchDelegate {
    var onMatch: OnMatchClosure?
    init(_ onMatch: @escaping OnMatchClosure) {
        self.onMatch = onMatch
    }
}
```

Create an instance of MatchHandler class and add it to the match delegates:
```swift
let matchHandler = MatchHandler.init({ matches, device in
    print(matches.description)
    print(device)
})
MatchMore.matchDelegates.add(matchHandler)
```

In this example, for every match there will be a message printed in the console describing all the received matches and their related device.
