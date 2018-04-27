---
title: Proximity Detection
sections:
  - Quickstart
---

{: #quickstart}
### Quickstart
Quickly create proximity detection for any types of `Device`.

Follow these four steps to create proximity detection:
1. Create a device to **feature proximity detection**
2. Create a publication to **broadcast the presence of a device**
3. Create a subscription to **start discovering near devices**
4. **Handle Match**, when subscribers and publishers are within range of each other

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

#### STEP 4: **Handle Match**, when subscribers and publishers are within range of each other
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
