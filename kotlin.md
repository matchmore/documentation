---
title: Android SDK
---
Android Matchmore SDK is a contextualized publish/subscribe model which can be used to model any geolocated or proximity based mobile application. Save time and make development easier by using our SDK. We are built on Android Location services and we also provide iBeacons compatibility.

The `Matchmore` is a static wrapper that provides you all the functions you need to use our SDK.
You can access default instance of matchmore by `Matchmore.instance`.

<p class="text-center">
This sdk is provided as open-source software:
</p>

<p class="text-center">
[*&nbsp;*{: .fa .fa-external-link} Android SDK on Github](https://github.com/matchmore/android-sdk){: .btn .btn-blue .btn-cta}
</p>

<p class="text-center">
SDK is written using Kotlin 1.2.
Android Matchmore SDK requires Android 4.4+
</p>

# Kotlin
* [Get Started](#kotlin-get-started)
1. [Gradle](#kotlin-gradle)
2. [Maven](#kotlin-maven)
3. [Quickstart example](#kotlin-quickstart-example)
* [Configuration](#kotlin-configuration)
1. [Request permission for Location Services](#kotlin-request-permission-for-location-services)
2. [Add SDK to the project](#kotlin-add-sdk-to-the-project)
3. [Start/Stop location updates](#kotlin-start-stop-location-updates)
4. [Configure custom location manager](#kotlin-configure-custom-location-manager)
* [Tutorials](#kotlin-tutorials)
1. [Create a Mobile Device](#kotlin-create-a-mobile-device)
2. [Create a Pin Device](#kotlin-create-a-pin-device)
3. [Start/Stop Monitoring for a device](#kotlin-start-stop-monitoring-for-a-device)
4. [Google Firebase Cloud notification](#kotlin-google-firebase-cloud-notification)
5. [WebSocket](#kotlin-websocket)
6. [Polling](#kotlin-polling)
7. [Publish](#kotlin-publish)
8. [Subscribe](#kotlin-subscribe)
9. [Get Matches](#kotlin-get-matches)
10. [Local States request](#kotlin-local-states-request)

{: #kotlin-get-started}
### Get Started

You can install Matchmore SDK using standard Android package managers:

{: #kotlin-gradle}
#### Gradle
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
```
implementation 'com.github.matchmore.alps-android-sdk:sdk:<latest version>'
```

Rx Wrapper
```
implementation 'com.github.matchmore.alps-android-sdk:rx:<latest version>'
```

{: #kotlin-maven}
##### Maven

```xml
<repositories>
		<repository>
		    <id>jitpack.io</id>
		    <url>https://jitpack.io</url>
		</repository>
</repositories>
```

SDK
```xml
<dependency>
    <groupId>com.github.matchmore.alps-android-sdk</groupId>
    <artifactId>sdk</artifactId>
    <version>latest version</version>
</dependency>
```

Rx Wrapper
```xml
<dependency>
    <groupId>com.github.matchmore.alps-android-sdk</groupId>
    <artifactId>rx</artifactId>
    <version>latest version</version>
</dependency>
```

{: #kotlin-quickstart-example}
#### Quickstart example
For Kotlin Developers:
```kotlin
package io.matchmore.sdk.example

import android.Manifest
import android.annotation.SuppressLint
import android.os.Bundle
import android.support.v7.app.AppCompatActivity
import android.util.Log
import android.widget.Toast
import com.gun0912.tedpermission.PermissionListener
import com.gun0912.tedpermission.TedPermission
import io.matchmore.config.SdkConfigTest
import io.matchmore.sdk.Matchmore
import io.matchmore.sdk.MatchmoreSDK
import io.matchmore.sdk.api.models.Publication
import io.matchmore.sdk.api.models.Subscription


class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        Matchmore.config(this, "Your-api-key", true)
        Matchmore.instance.apply {
            startUsingMainDevice(
                    { device ->
                        Log.i(TAG, "start using device ${device.name}")

                        val publication = Publication("Test Topic", 20.0, 100.0)
                        publication.properties = hashMapOf("test" to "true")
                        createPublicationForMainDevice(publication, { result ->
                            Log.i(TAG, "Publication created ${result.topic}")
                        }, Throwable::printStackTrace)

                        matchMonitor.addOnMatchListener { matches, _ ->
                            Log.i(TAG, "Matches found: ${matches.size}")
                        }
                        createPollingSubscription()
                        checkLocationPermission()
                    }, Throwable::printStackTrace)
        }
    }

    override fun onResume() {
        super.onResume()
        Matchmore.instance.matchMonitor.startPollingMatches()
    }

    override fun onPause() {
        super.onPause()
        Matchmore.instance.matchMonitor.stopPollingMatches()
    }

    override fun onDestroy() {
        super.onDestroy()
        Matchmore.instance.apply {
            stopRanging()
            stopUpdatingLocation()
        }
    }

    private fun MatchmoreSDK.createSocketSubscription() {
        val subscription = Subscription("Test Topic", 20.0, 100.0)
        subscription.selector = "test = 'true'"
				subscription.pushers = mutableListOf("ws")
        createSubscriptionForMainDevice(subscription, { result ->
            Log.i(TAG, "Subscription created ${result.topic}")
        }, Throwable::printStackTrace)
    }

    private fun MatchmoreSDK.createPollingSubscription() {
        val subscription = Subscription("Test Topic", 20.0, 100.0)
        subscription.selector = "test = 'true'"
        createSubscriptionForMainDevice(subscription, { result ->
            Log.i(TAG, "Subscription created ${result.topic}")
        }, Throwable::printStackTrace)
    }

    private fun MatchmoreSDK.createFcmSubscription(deviceToken: String) {
        val subscription = Subscription("Test Topic", 20.0, 100.0)
        subscription.selector = "test = 'true'"
				subscription.pushers = mutableListOf("fcm://" + deviceToken)
        createSubscriptionForMainDevice(subscription, { result ->
            Log.i(TAG, "Subscription created ${result.topic}")
        }, Throwable::printStackTrace)
    }

    private fun checkLocationPermission() {
        val permissionListener = object : PermissionListener {
            @SuppressLint("MissingPermission")
            override fun onPermissionGranted() {
                Matchmore.instance.apply {
                    startUpdatingLocation()
                    startRanging()
                }
            }

            override fun onPermissionDenied(deniedPermissions: ArrayList<String>) {
                Toast.makeText(this@MainActivity, R.string.if_you_reject, Toast.LENGTH_SHORT).show()
            }
        }
        TedPermission.with(this)
                .setPermissionListener(permissionListener)
                .setDeniedMessage(R.string.if_you_reject)
                .setPermissions(Manifest.permission.ACCESS_FINE_LOCATION)
                .check()
    }

    companion object {
        private const val TAG = "MatchMore"
    }
}
```

{: #kotlin-configuration}
### Configuration
{: #kotlin-request-permission-for-location-services}
#### Request permission for Location Services
Depending on your needs, your app may require Location Services to work in the background, which means we need to set up Location Services usage description. This description will be shown to the user when asking them about allowing the app to access their location.

Users must grant permission for an app to access personal information, including the current location. Although people appreciate the convenience of using an app that has access to this information, they also expect to have control over their private data.

In the project navigator, find the file AndroidManifest.xml (App/manifests), double click on it. Then, inside <manifest> tag, add the following lines:

```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```


When opening app for the first time, the system will prompt `authorization alert` with the filled description. Users can decide wether to accept or reject the request permission.

<div class="callout-block callout-success"><div class="icon-holder">*&nbsp;*{: .fa .fa-thumbs-up}
</div><div class="content">
{: .callout-title}
#### Good practice
Request permission at launch only when necessary for your app to function. Users won’t be bothered by this request if it’s obvious that your app depends on their personal information to operate. For example, an app might only request access to the current location when activating a location tracking feature.

</div></div>

{: #kotlin-add-sdk-to-the-project}
#### Add SDK to the project
Matchmore SDKs are very malleable. The SDK is configured by default to work for general cases. But it is possible to configure it according to your needs.
First set up Matchmore SDK Cloud credentials, this will allow the SDK to communicate with Matchmore Cloud on your behalf.

You can generate a token API-key for yourself on matchmore.io.
When you have your token, create your `MatchMoreConfig` with your API-key.

```kotlin
// MatchMoreConfig is a structure that defines all variables needed to configure MatchMore SDK.
data class MatchmoreConfig(
        var context: Context,
        val apiKey: String,
        var debugLog: Boolean
) {

    init {
        context = context.applicationContext
    }
}
```

Starting with a basic setup. Use MatchmoreSDK with default configuration.

Basic configuration:
```kotlin
class ExampleApp : MultiDexApplication() {
    override fun onCreate() {
        super.onCreate()
        Matchmore.config(this, "YOUR_API_KEY", true)
    }
}
```

{: #kotlin-start-stop-location-updates}
#### Start/Stop location updates
```
private fun checkLocationPermission() {
        val permissionListener = object : PermissionListener {
            @SuppressLint("MissingPermission")
            override fun onPermissionGranted() {
                Matchmore.instance.apply {
                    startUpdatingLocation() // <- Start location updates
										stopUpdatingLocation() // <- Stop location udpates
                }
            }

            override fun onPermissionDenied(deniedPermissions: ArrayList<String>) {
                Toast.makeText(this@MainActivity, R.string.if_you_reject, Toast.LENGTH_SHORT).show()
            }
        }
        TedPermission.with(this)
                .setPermissionListener(permissionListener)
                .setDeniedMessage(R.string.if_you_reject)
                .setPermissions(Manifest.permission.ACCESS_FINE_LOCATION)
                .check()
    }
```

{: #kotlin-configure-custom-location-manager}
#### Configure custom location manager
As you already saw `MatchmoreConfig` plays key role in configuring the SDK, though there're more custom integrations you can do.
For example it's possible to entirely change location provider:

```kotlin
val locationProvider = object : MatchmoreLocationProvider {
	override fun startUpdatingLocation(sender: LocationSender) {
		sender.sendLocation(MatchmoreLocation(createdAt = System.currentTimeMillis(), latitude = 80.0, longitude = 80.0))
	}
	override fun stopUpdatingLocation() { }
}
// update location
matchMoreSdk.startUpdatingLocation(locationProvider)
```

{: #kotlin-tutorials}
### Tutorials

{: #kotlin-create-a-mobile-device}
#### Create a Mobile Device
```kotlin
Matchmore.instance.startUsingMainDevice(
                { device ->
                    Log.i(TAG, "start using device ${device.name}")
                }, Throwable::printStackTrace)
```

{: #kotlin-create-a-pin-device}
#### Create a Pin Device
```kotlin
val pinDevice = PinDevice("New Pin Device", MatchmoreLocation(latitude = 46.519962, longitude = 6.633597))
Matchmore.instance.devices.create(pinDevice, { device ->
	Log.i(TAG, "Created Pin Device: ${device.id}")
}, Throwable::printStackTrace)
```

{: #kotlin-start-stop-monitoring-for-a-device}
#### Start/Stop Monitoring for a device

```kotlin
var pinDevice = PinDevice("Pin Device", location = MatchmoreLocation(latitude = 46.519962, longitude = 6.633597))

Matchmore.instance.createPinDevice(pinDevice, { device ->
		Log.i(TAG, "Created Pin Device: ${device.name}")
		Matchmore.instance.matchMonitor.startMonitoringFor(device) // <- Start the monitoring of this pin device
}, Throwable::printStackTrace)
```

{: #kotlin-google-firebase-cloud-notification}
#### Google Firebase Cloud Notification

To configure push notifications you need to call this code. All token will be propagated to subscriptions created by main device that have `fcm://` in pushers.

```kotlin
val token = "FCM_TOKEN"
Matchmore.instance.registerDeviceToken(token)
```

{: #kotlin-websocket}
#### WebSocket

The other option of getting matches from Matchmore server is a web socket. Note that socket is alternative for push notifications, but it's more energy consuming. The biggest advantage of using it is permission less configuration. Though this solution will work only when application is active. To make web socket work your subscription's selector has to contain `ws`. Then you can open socket connection by calling:
```kotlin
// Opening socket
Matchmore.instance.matchMonitor.openSocketForMatches()

// Closing socket
Matchmore.instance.matchMonitor.closeSocketForMatches()
```

{: #kotlin-polling}
#### Polling

Use these functions to start or stop polling matches from Matchmore Cloud.

```kotlin
// Start polling matches every 1000 miliseconds
Matchmore.instance.matchMonitor.startPollingMatches(1000)
// Stop polling
Matchmore.instance.matchMonitor.stopPollingMatches()
```

{: #kotlin-publish}
#### Publish
Creating publication on main device:
```kotlin
val publication = Publication("Test Topic", 1000.0, 1000.0)
publication.properties = hashMapOf("isGreen" to true, "name" to "A New Pub")
createPublicationForMainDevice(publication, { result ->
	Log.i(TAG, "Publication created ${result.topic}")
}, Throwable::printStackTrace)
```

{: #kotlin-subscribe}
#### Subscribe
Creating subscription on a pin device:
```kotlin
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

{: #kotlin-get-matches}
#### Get Matches
Start and stop listening for new matches by adding listener:
```kotlin
val listener = { matches, device ->
	Log.i(TAG, "Matches found: ${matches.size} on device: ${device}")
}
Matchmore.instance.matchMonitor.addOnMatchListener(listener)

// remove when not needed anymore
Matchmore.instance.matchMonitor.removeOnMatchListener(listener)
```

{: #kotlin-local-states-request}
#### Local States request
```swift
// You can access locally cached subscriptions, publications and devices
Matchmore.instance.subscriptions
Matchmore.instance.publications
Matchmore.instance.devices
Matchmore.instance.knownBeacons

// you can delete also your sub, pub and device by calling:
Matchmore.instance.subscriptions.delete(sub)
Matchmore.instance.publications.delete(pub)
Matchmore.instance.devices.delete(pin)
Matchmore.instance.devices.delete(mobile)
```
