---
title: Unity 3D SDK
---
<div class="callout-block callout-warning"> <div class="icon-holder"> <p><em class="fa fa-info-circle">&nbsp;</em></p> </div> <div class="content"> <h4 class="callout-title" id="questions-">Unity vs .NET</h4> <p>Just recently Unity released support for newer version of C# in Unity 2018. We are working on making the .NET work with Unity. But for now you are free to use this version of the SDK</p> </div> </div>

* [Get Started](#unity-get-started)
1. [Installation](#unity-installation)
2. [Quickstart example](#unity-quickstart-example)
* [Tutorials](#unity-tutorials)
1. [Start/stop Matchmore](#unity-start-stop)
2. [Start/Stop updating location](#unity-start-stop-location)
3. [Manual update locations](#unity-manual-update)
5. [Publish](#unity-publish)
6. [Subscribe](#unity-subscribe)
7. [Get matches](#unity-get-matches)
8. [Polling](#unity-polling)
9. [WebSocket](#unity-websocket)
10. [Matches from static instance](#unity-matches-globally)
11. [Local States request](#unity-local-states-request)
* [TLS 1.2 on Unity](#unity-tls)
* [Demo](#unity-demo)


{: #unity-installation}
## Installation

Clone and copy contents *Assets/Matchmore* of the repository into *Assets/Matchmore*. You can change the target folder

{: #unity-quickstart-example}
## Quickstart sample

```csharp
        //make sure new TLS is enabled, this is only available in .NET 4.x Equivalent
        System.Net.ServicePointManager.SecurityProtocol = System.Net.SecurityProtocolType.Tls12 | System.Net.SecurityProtocolType.Tls11;

        //get the api key from our portal
        var config = Matchmore.Config.WithApiKey(apiKey);
        Matchmore.Configure(config);
	    // we register the device as the main device
        var mainDevice = Matchmore.Instance.MainDevice;

        //methods interacting with devices by default use the main device, but there are overloads to provide other devices
        var pub = Matchmore.Instance.CreatePublication(new Publication
        {
            Topic = "Unity",
            Duration = 30,
            Range = 100,
            Properties = new Dictionary<string, object>(){
                {"test", true},
                {"price", 199}
            }
        });

        var sub = Matchmore.Instance.CreateSubscription(new Subscription
        {
            Topic = "Unity",
            Duration = 30,
            Range = 100,
            Selector = "test = true and price <= 200",
            Pushers = new List<string>() { "ws" }
        });

        foreach (var _pub in Matchmore.Instance.ActivePublications)
        {
            LogLine(string.Format("Pub {0} existings for another {1} seconds", _pub.Id, _pub.SecondsRemaining()));
        }

        foreach (var _sub in Matchmore.Instance.ActiveSubscriptions)
        {
            LogLine(string.Format("Sub {0} existing for another {1} seconds", _sub.Id, _sub.SecondsRemaining()));
        }

        //this method is available but we run a coroutine in the background which starts the unity location service and runs this call every time the location was changed
        Matchmore.Instance.UpdateLocation(new Location
        {
            Latitude = 54.414662,
            Longitude = 18.625498
        });

        //query for available matches via a single request
        var matches = Matchmore.Instance.GetMatches();
        matches.ForEach(m =>
        {
            LogLine("Got Match " + m.Id + " single call");
        });

        //creates a monitor on a single device which has an event handler(by default the main one)
        var socketMonitor = Matchmore.Instance.SubscribeMatches(Matchmore.MatchChannel.Websocket);

        //creates a monitor on a single device which has an event handler(by default the main one)
        var pollingMonitor = Matchmore.Instance.SubscribeMatches(Matchmore.MatchChannel.Polling);

        //this event handler fires only for a single device
        socketMonitor.MatchReceived += (sender, e) => {
            LogLine(string.Format("Received match {0} from device {1} with channel {2} directly from monitor", e.Matches[0].Id, e.Device.Id, e.Channel));
        };

        pollingMonitor.MatchReceived += (sender, e) => {
            LogLine(string.Format("Received match {0} from device {1} with channel {2} directly from monitor", e.Matches[0].Id, e.Device.Id, e.Channel));
        };

        //this event handler fires for all devices created
        Matchmore.Instance.MatchReceived += (sender, e) => {
            LogLine(string.Format("Received match {0} from device {1} with channel {2}", e.Matches[0].Id, e.Device.Id, e.Channel));
        };

```

{: #unity-tutorials}
### Tutorials
{: #unity-start-stop}
#### Start/stop Matchmore
```csharp
//Reset will stop monitors and location updates
Matchmore.Reset();
```
{: #unity-start-stop-location}
#### Start/Stop updating location
```csharp
Matchmore.Instance.StartLocationService();
```

{: #unity-manual-update}
##### Manual update locations

```csharp
Matchmore.Instance.UpdateLocation(new Location
        {
            Latitude = 54.414662,
            Longitude = 18.625498
        });

```

{: #unity-mobile-device}
#### Create a Mobile Device
```csharp
var pubDevice = Matchmore.Instance.CreateDevice(new MobileDevice
{
    Name = "Publisher"
});

{: #unity-pin-device}
```
#### Create a Pin Device
```csharp
var pubDevice = Matchmore.Instance.CreateDevice(new PinDevice
{
    Name = "Publisher"
});
```

{: #unity-publish}
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
var createdPub = Matchmore.Instance.CreatePublication(pub, pubDevice);
```

{: #unity-subscribe}
#### Subscribe
```csharp
var sub = new Subscription
        {
            Topic = "Test Topic",
            Duration = 30,
            Range = 100,
            Selector = "test = true and price <= 200"
        };

var createdSub = Matchmore.Instance.CreatePublicationCreateSubscription(sub);
```

{: #unity-get-matches}
#### GetMatches
```csharp
//you can call to get matches at your leisure, if you don't want to use the monitors. In the end the monitors are calling exactly this method
var matches = Matchmore.Instance.GetMatches();
```

{: #unity-polling}
#### Polling
```csharp
var pollingMonitor = Matchmore.Instance.SubscribeMatches(Matchmore.MatchChannel.polling);
pollingMonitor.MatchReceived += (sender, e) => {
            LogLine(string.Format("Received match {0} from device {1} with channel {2} directly from monitor", e.Matches[0].Id, e.Device.Id, e.Channel));
        };
```

{: #unity-websocket}
#### Websocket

```csharp
var socketMonitor = Matchmore.Instance.SubscribeMatches(Matchmore.MatchChannel.websocket);
socketMonitor.MatchReceived += (sender, e) => {
            LogLine(string.Format("Received match {0} from device {1} with channel {2} directly from monitor", e.Matches[0].Id, e.Device.Id, e.Channel));
        };
```

{: #unity-matches-globally}
#### Matches from static instance
```csharp
Matchmore.Instance.MatchReceived += (sender, e) => {
    LogLine(string.Format("Received match {0} from device {1} with channel {2}", e.Matches[0].Id, e.Device.Id, e.Channel));
};
```

{: #unity-local-states-request}
#### Local States request
```csharp
foreach (var _pub in Matchmore.Instance.ActivePublications)
{
    LogLine(string.Format("Pub {0} existings for another {1} seconds", _pub.Id, _pub.SecondsRemaining()));
}

foreach (var _sub in Matchmore.Instance.ActiveSubscriptions)
{
    LogLine(string.Format("Sub {0} existing for another {1} seconds", _sub.Id, _sub.SecondsRemaining()));
}
```

{: #unity-tls}
## TLS 1.1 and 1.2

We have moved to support newer cyphers as Apple stopped support most of TLS 1.0 cyphers. Some version of Unity might not support this but here is a trick which might make it work.

1. Set Scripting Runtime Version to *.NET 4.x Equivalent*
If this didn't work
2. Unzip the [SystemSecurity.zip](https://github.com/matchmore/unity-sdk/raw/master/SystemSecurity.zip) into a folder called *plugins* which can exist anywhere in the solution.

{: #unity-demo}
## Demo
A Unity demo is available in our [Unity github](https://github.com/matchmore/unity-sdk/blob/master/Assets/Matchmore/Example/MatchmoreExample.cs).

