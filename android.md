---
title: Android installation & configuration
---

{: #android}
### Android

Android Matchmore SDK is a contextualized publish/subscribe model which can be used to model any geolocated or proximity based mobile application. Save time and make development easier by using our SDK. We are built on Android Location services and we also provide iBeacons compatibility. 

The `Matchmore` is a static wrapper that provides you all the functions you need to use our SDK.
You can access default instance of matchmore by `Matchmore.instance`.

SDK is written using Kotlin 1.2.
Android Matchmore SDK requires Android 4.4+

#### Standard Integration

You can install Matchmore SDK using standard Android package managers:

##### Gradle
Add it in your root build.gradle at the end of repositories:

```
allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
}
```

SDK
```implementation 'com.github.matchmore.alps-android-sdk:sdk:<latest version>'```

Rx Wrapper
```implementation 'com.github.matchmore.alps-android-sdk:rx:<latest version>'```

##### Maven
```XML
<repositories>
		<repository>
		    <id>jitpack.io</id>
		    <url>https://jitpack.io</url>
		</repository>
</repositories>
```

SDK
```XML
<dependency>
    <groupId>com.github.matchmore.alps-android-sdk</groupId>
    <artifactId>sdk</artifactId>
    <version>latest version</version>
</dependency>
```

Rx Wrapper

```XML
<dependency>
    <groupId>com.github.matchmore.alps-android-sdk</groupId>
    <artifactId>rx</artifactId>
    <version>latest version</version>
</dependency>
```

After getting dependecy you can start using Matchmore functionalities by simply calling `startUsingMainDevice()` function - it will create mobile device object that will be treated as main mobile device. You can also create other types of devices such as pin or beacon. After device is ready you can easily create publications and subscriptions in order to get matches. Take a look at examples below for most common use cases:

Basic configuration:
```Kotlin
class ExampleApp : MultiDexApplication() {
    override fun onCreate() {
        super.onCreate()
        Matchmore.config(this, "YOUR_API_KEY", true)
    }
}
```

```Java
```

Creating main device:
```Kotlin
Matchmore.instance.startUsingMainDevice(
                { device ->
                    Log.i(TAG, "start using device ${device.name}")
                }, Throwable::printStackTrace)
```

```Java
```

Creating pin device:
```Kotlin
val pinDevice = PinDevice("New Pin Device", MatchmoreLocation(latitude = 46.519962, longitude = 6.633597))
Matchmore.instance.devices.create(pinDevice, { device ->
	Log.i(TAG, "Created Pin Device: ${device.id}")
}, Throwable::printStackTrace)
```

```Java
```

Start and stop listening for new matches by adding listener:
```Kotlin
val listener = { matches, device ->
	Log.i(TAG, "Matches found: ${matches.size} on device: ${device}")
}
Matchmore.instance.matchMonitor.addOnMatchListener(listener)

// remove when not needed anymore
Matchmore.instance.matchMonitor.removeOnMatchListener(listener)
```

```Java
```

Creating publication on main device:
```Kotlin
val publication = Publication("Test Topic", 1000.0, 1000.0)
publication.properties = hashMapOf("isGreen" to true, "name" to "A New Pub")
createPublicationForMainDevice(publication, { result ->
	Log.i(TAG, "Publication created ${result.topic}")
}, Throwable::printStackTrace)
```

```Java
```

Creating subscription on a pin device:
```Kotlin
val subscription = Subscription("Test Topic", 1.0, 0.0)
subscription.selector = "isGreen = true" // Getting matches will all publication that have isGreen property set to true
// Note:
// "ws" is websocket hook
// "fcm://" is push notification hook
subscription.pushers = mutableListOf("ws", "fcm://")
createSubscription(subscription, pinDeviceId, { result ->
	Log.i(TAG, "Subscription created ${result.topic}")
}, Throwable::printStackTrace)
```

```Java
```
#### Custom Integration

As you already saw `MatchmoreConfig` plays key role in configuring the SDK, though there're more custom integrations you can do.
For example it's possible to entirely change location provider:

```Kotlin
val locationProvider = object : MatchmoreLocationProvider {
	override fun startUpdatingLocation(sender: LocationSender) {
		sender.sendLocation(MatchmoreLocation(createdAt = System.currentTimeMillis(), latitude = 80.0, longitude = 80.0))
	}
	override fun stopUpdatingLocation() { }
}
// update location
matchMoreSdk.startUpdatingLocation(locationProvider)
```

```Java
```
#### Google Firebase Cloud Notification

To configure push notifications you need to call this code. All token will be propgated to subscriptions created by main device that have `fcm://` in pushers.

```Kotlin
val matchMoreSdk = Matchmore.instance as AlpsManager
val token = "FCM_TOKEN"
Matchmore.instance.registerDeviceToken(token)
```

```Java
```

#### Web Socket

The other option of getting matches from Matchmore server is a web socket. Note that socket is alternative for push notifications, but it's more energy consuming. The biggest advantage of using it is permissionless configuration. Though this solution will work only when application is active. To make web socket work your subscription's selector has to contain `ws`. Then you can open socket connection by calling:
```Kotlin
// Opening socket
Matchmore.instance.matchMonitor.openSocketForMatches()

// Closing socket
Matchmore.instance.matchMonitor.closeSocketForMatches()
```

```Java
```
#### Polling

Use these functions to start or stop polling matches from Matchmore Cloud.

```Kotlin
// Start polling matches every 1000 miliseconds
Matchmore.instance.matchMonitor.startPollingMatches(1000)
// Stop polling
Matchmore.instance.matchMonitor.stopPollingMatches()
```

```Java
```
