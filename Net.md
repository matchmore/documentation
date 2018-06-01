---
title: .NET SDK
---

### Getting started
#### NuGet package
You can integrate Matchmore to your project via NuGet.

A NuGet-walkthrough can be found here  [https://docs.microsoft.com/en-us/visualstudio/mac/nuget-walkthrough](https://docs.microsoft.com/en-us/visualstudio/mac/nuget-walkthrough)

### Configuration
#### Configure project (Request permission)
##### iOS info.plist
In iOS 10, developers have to declare ahead of time any access to a user’s private data in their Info.plist.

You must add the follow privacy settings into the Info.plist file to get access to location data:

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
##### Android request permission
In Android, you need to add the following lines of code in your MainActivity.cs file inside of onCreate() function:
```Java
            const string p1 = Android.Manifest.Permission.AccessFineLocation;
            const string p2 = Android.Manifest.Permission.AccessCoarseLocation;
            RequestPermissions(new string[] { p1, p2 }, 0);
```

Also, find your AndroidManifest.xml file in the properties folder and tick the followings:
Required permissions :
* ACCESS_COARSE_LOCATION
* ACCESS_FINE_LOCATION
#### Add SDK to project

Setup application API key and world, get it for free from [http://matchmore.io/](http://matchmore.io/).

```csharp
async Task SetupMatchmore()
{
    await Matchmore.SDK.Matchmore.ConfigureAsync("YOUR_API_KEY");
}
```

#### QuickStart
Create main mobile device, publication and subscription. Please note that we're not caring about errors right now.

```csharp
var tsk = Task.Run(async () =>
            {
                // Configure Matchmore with your credentials
                await Matchmore.SDK.Matchmore.ConfigureAsync("YOUR_API_KEY");

                //you can access the device later by calling Matchmore.SDK.Matchmore.Instance.MainDevice
                await Matchmore.SDK.Matchmore.Instance.SetupMainDeviceAsync();

                // Create the subscription
                var sub = new Subscription
                {
                    Topic = "Test Topic",
                    Duration = 30,
                    Range = 100,
                    Selector = "test = true and price <= 200"
                };

                // Start the creation of your subscription in Matchmore service
                var createdSub = await Matchmore.SDK.Matchmore.Instance.CreateSubscriptionAsync(sub);
                System.Diagnostics.Debug.WriteLine($"{createdSub.CreatedAt}");

                var pubDevice = await Matchmore.SDK.Matchmore.Instance.CreateDeviceAsync(new MobileDevice
                {
                    Name = "Publisher"
                });

                var pub = new Publication
                {
                    Topic = "Test Topic",
                    Duration = 30,
                    Range = 100,
                    Properties = new Dictionary<string, object>(){
                {"test", true},
                {"price", 199}
            }
                };
                //you can pass explicitly the device you would want to use
                var createdPub = await Matchmore.SDK.Matchmore.Instance.CreatePublicationAsync(pub);
                System.Console.WriteLine($"created publication {createdPub.Id}");

                // To receive matches, you need to create a monitor
                //default params of .SubscribeMatches() use your main device and Polling as a channel delivery mechanism
                var monitor = Matchmore.SDK.Matchmore.Instance.SubscribeMatches();
                monitor.MatchReceived += (object sender, MatchReceivedEventArgs e) => {
                    //handle your match
                };

                //if  you don't have access to your monitor, you can attach the event handler on the Matchmore Instance
                Matchmore.SDK.Matchmore.Instance.MatchReceived += (object sender, MatchReceivedEventArgs e) => {
                    //handle your match, the sender will be your monitor
                };
            }

                    );
            Task.WaitAll(tsk);
```
#### Start/stop Matchmore
#### Start/Stop updating location
#### For iOS/Android : Configure LocationManager
##### Foreground update locations
##### Background update locations
##### Manual update locations
### Tutorials
#### Create a Mobile Device
```csharp
            var pubDevice = await Matchmore.SDK.Matchmore.Instance.CreateDeviceAsync(new MobileDevice
            {
                Name = "Publisher"
            });
```
#### Create a Pin Device
#### Create a Beacon Device (Except for SDKs that don’t support beacons yet)
#### Start/Stop Monitoring for device, APNS/FCM, WebSocket and Polling
#### Publish
```csharp
var pub = new Publication
		{
			Topic = "Test Topic",
			Duration = 30,
			Range = 100,
			Properties = new Dictionary<string, object>(){
				{"test", true},
				{"price", 199}
			}
		};
//you can pass explicitly the device you would want to use
var createdPub = await Matchmore.SDK.Matchmore.Instance.CreatePublicationAsync(pub, pubDevice);
```
#### Subscribe
```csharp
var sub = new Subscription
        {
            Topic = "Test Topic",
            Duration = 30,
            Range = 100,
            Selector = "test = true and price <= 200"
        };

var createdSub = await Matchmore.SDK.Matchmore.Instance.CreateSubscriptionAsync(sub);
```
#### GetMatches
#### Third party match providers (APNS and FCM)

To use your match provider of your choice, you need to wire it differently depending on the platform.

First you need to provide the token for the platform (FCM or APNS) to Matchmore so we know where to route the match id

```csharp
//fcm
Matchmore.SDK.Matchmore.Instance.UpdateDeviceCommunicationAsync(new Matchmore.SDK.Communication.FCMTokenUpdate("token taken from FCM"));
//basing of https://docs.microsoft.com/en-us/xamarin/android/data-cloud/google-messaging/remote-notifications-with-fcm?tabs=vswin

[Service]
[IntentFilter(new[] { "com.google.firebase.INSTANCE_ID_EVENT" })]
public class MyFirebaseIIDService : FirebaseInstanceIdService
{
    const string TAG = "MyFirebaseIIDService";
    public override void OnTokenRefresh()
    {
        var refreshedToken = FirebaseInstanceId.Instance.Token;
        Log.Debug(TAG, "Refreshed token: " + refreshedToken);
        Matchmore.SDK.Matchmore.Instance.UpdateDeviceCommunicationAsync(new Matchmore.SDK.Communication.FCMTokenUpdate(refreshedToken));
    }
}

//apns
Matchmore.SDK.Matchmore.Instance.UpdateDeviceCommunicationAsync(new Matchmore.SDK.Communication.APNSTokenUpdate("token taken from APNS"));
//basing of https://github.com/xamarin/ios-samples/blob/master/Notifications/AppDelegate.cs
public override void RegisteredForRemoteNotifications (UIApplication application, NSData deviceToken)
{
	Matchmore.SDK.Matchmore.Instance.UpdateDeviceCommunicationAsync(new Matchmore.SDK.Communication.APNSTokenUpdate(deviceToken.ToString()));
}

```

Then you need to tell Matchmore whenever you will get a match id to retrieve.

```csharp
//android fcm
[Service]
[IntentFilter(new[] { "com.google.firebase.MESSAGING_EVENT" })]
public class MyFirebaseMessagingService : FirebaseMessagingService
{

    IMatchProviderMonitor matchProviderMonitor = Matchmore.SDK.Matchmore.Instance.SubscribeMatchesWithThirdParty();
    const string TAG = "MyFirebaseMsgService";
    public override void OnMessageReceived(RemoteMessage message)
    {
        matchProviderMonitor.ProvideMatchIdAsync(MatchId.Make(message.GetNotification().Body));
    }
}

//ios apns in your AppDelegate
public override void ReceivedRemoteNotification (UIApplication application, NSDictionary userInfo)
{
    IMatchProviderMonitor matchProviderMonitor = Matchmore.SDK.Matchmore.Instance.SubscribeMatchesWithThirdParty();
    var matchId = userInfo["matchId"].ToString();
    matchProviderMonitor.ProvideMatchIdAsync(MatchId.Make(matchId));
}
```

Additional info you might find useful
* [how to setup APNS](https://github.com/matchmore/alps-ios-sdk/blob/master/ApnsSetup.md).
* [fcm in xamarin](https://docs.microsoft.com/en-us/xamarin/android/data-cloud/google-messaging/remote-notifications-with-fcm?tabs=vswin)
* [apns in xamarin](https://docs.microsoft.com/en-us/xamarin/ios/platform/user-notifications/deprecated/remote-notifications-in-ios)

#### Local CRUD request (Create, Find, FindAll, Delete and DeleteAll)
### Development

To run tests for .NET Standard use `dotnet` command line utility
```
dotnet test SDK.Tests/SDK.NetStandard.Tests.csproj
```

Other devices can be run from visual studio

### Changelog
### Supported Platform
