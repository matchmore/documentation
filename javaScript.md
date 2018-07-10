---
title: JavaScript SDK
---
<p class="text-center">
This sdk is provided as open-source software:
</p>

<p class="text-center">
[*&nbsp;*{: .fa .fa-external-link} Javascript SDK on Github](https://github.com/matchmore/js-sdk){: .btn .btn-blue .btn-cta}
</p>

* [Get Started](#javascript-get-started)
1. [NPM](#javascript-npm)
2. [Yarn](#javascript-yarn)
3. [HTML document](#javascript-html-document)
4. [Quickstart](#javascript-quickstart-example)
5. [Get the API key](#javascript-get-the-api-key)
6. [Web Setup](#javascript-web-setup)
7. [React Native Setup](#javascript-react-native-setup)
8. [Quickstart example](#javascript-example)
* [Configuration](#javascript-configuration)
1. [Request permission for Location Services](#javascript-request-permission-for-location-services)
2. [Add SDK to the project](#javascript-add-sdk-to-the-project)
3. [Start/Stop location updates](#javascript-start-stop-location-updates)
4. [Configure custom location manager](#javascript-configure-custom-location-manager)
5. [Manual update locations](#javascript-manual-update-locations)
* [Tutorials](#javascript-tutorials)
1. [Create a Mobile Device](#javascript-create-a-mobile-device)
2. [Create a Pin Device](#javascript-create-a-pin-device)
3. [Start/Stop Monitoring for a device](#javascript-start-stop-monitoring-for-a-device)
4. [WebSocket](#javascript-websocket)
5. [Polling](#javascript-polling)
6. [Publish](#javascript-publish)
7. [Subscribe](#javascript-subscribe)
8. [Get Matches](#javascript-get-matches)
9. [Local States request](#javascript-local-states-request)
* [Demo](#javascript-demo)

{: #javascript-get-started}
### Get started

{: #javascript-npm}
#### NPM
In a project backed by npm (like react) you can just type

```
npm install @matchmore/matchmore --save
```

{: #javascript-yarn}
#### Yarn
To include Matchmore SDK in your project you need to use Yarn.

```
yarn add @matchmore/matchmore
```

{: #javascript-html-document}
#### HTML Document
In a HTML document

```html
<script src="../../dist/web/matchmore.js"></script>
```

{: #javascript-quickstart-example
### Quickstart example

This is an example for web usage of the matchmore JavaScript SDK

{: #javascript-get-the-api-key}
#### Get the API key

Setup application API key and world, get it for free from [matchmore.io/](https://matchmore.io/).

And then start your application with minimum config, with in memory persistence

```javascript
import { Manager } from "matchmore";
//...
this.manager = new Manager(
  "<Your api key>"
)
```

More robust setup

```javascript
import { Manager, LocalStoragePersistenceManager } from "matchmore";
//...
const localPersistenceManager = new LocalStoragePersistenceManager();
await localPersistenceManager.load();
this.manager = new Manager(
  apiKey,
  undefined,
  localPersistenceManager,
  // undefined,
  {
    enableHighAccuracy: false,
    timeout: 60000,
    maximumAge: 60000
  }
)
```

{: #javascript-web-setup}
#### Web Setup
Web setup can done inline like this

```javascript
matchmore.PlatformConfig.storage = {
  save: (key, value) => window.localStorage.setItem(key, value),
  load: (key) => window.localStorage.getItem(key),
  remove: (key) => window.localStorage.removeItem(key),
}
```

{: #javascript-react-native-setup}
#### React Native Setup
React Native persistence requires some setup, best if put in its own file and import at the start of the application

```javascript
const { AsyncStorage } = require('react-native');

const save = async (key, value) => {
  await AsyncStorage.setItem(key, value);
}

const load = async (key) => {
  return await AsyncStorage.getKey(key);
}

const remove = async (key) => {
  return await  AsyncStorage.removeItem(key);
}

module.exports = {
  save,
  load,
  remove
}

```

{: #javascript-example}
#### Quickstart example
Here is an example with using promise chain syntax, you can freely use also the async/await pattern.

```javascript
 matchmore.PlatformConfig.storage = {
              save: (key, value) => window.localStorage.setItem(key, value),
              load: (key) => window.localStorage.getItem(key),
              remove: (key) => window.localStorage.removeItem(key),
            };
const localStoragePersistenceManager = new matchmore.LocalStoragePersistenceManager();
let manager = new matchmore.Manager(
  apiKey,
  undefined,
  localStoragePersistenceManager
);
manager.createMobileDevice("me", "browser", "").then(device => {
    manager.startMonitoringMatches();
    manager.onMatch = (match) => {
        writeToScreen(match.publication.properties.name + " matched!");
    };
    return device;
}).then(device => {
    //lets wait for the current location
    let location = new Promise(resolve => {
        manager.onLocationUpdate = (loc) => {
            writeToScreen("Got Location");
            resolve(loc);
        };
        manager.startUpdatingLocation();
    });
    location.then(location => {
      const { latitude, longitude, altitude } = location.coords;
      const coords = {
        latitude,
        longitude,
        altitude: altitude || 0,
      }
        let publication = manager.createPinDevice(
            "Our test pin",
            coords,
        ).then(pin => {
            let p1 = manager.createPublication(
                "my-topic",
                100 /* m */,
                20 /* s */,
                { "age": 20, "name": "Clara" },
                pin.id);
            let p2 = manager.createPublication(
                "my-topic",
                100 /* m */,
                20 /* s */,
                { "age": 18, "name": "Justine" },
                pin.id);
            let p3 = manager.createPublication(
                "my-topic",
                100 /* m */,
                20 /* s */,
                { "age": 17, "name": "Alex" },
                pin.id);
            return Promise.all([p1, p2, p3]);
        });
    }).then(_ => {
        let subscription = manager.createSubscription(
            "my-topic",
            100 /* m */,
            20 /* s */,
            "age >= 18"
        );
        return subscription;
    });
});
```

Developers are also free to use async/await syntax

```javascript
async multiplePublications(){
   await manager.createPublication(
       "my-topic",
       100 /* m */,
       20 /* s */,
       { "age": 20, "name": "Clara" },
       pin.id);
   await manager.createPublication(
       "my-topic",
       100 /* m */,
       20 /* s */,
       { "age": 18, "name": "Justine" },
       pin.id);
   await manager.createPublication(
       "my-topic",
       100 /* m */,
       20 /* s */,
       { "age": 17, "name": "Alex" },
       pin.id);
   return;
}
```

Developer is free to mix and match the styles

{: #javascript-request-permission-for-location-services}
#### Request permission for Location Services

The SDK assumes you gave access to location services, so you need to handle it gracefully and fitting in context of your app or let the framework yield a permission request for you.

{: #javascript-add-sdk-to-the-project}
#### Add SDK to the project

```javascript
import { Manager } from "matchmore";
//...
this.manager = new Manager(
  "<Your api key>"
)

```

{: #javascript-configuration}
### Configuration

More robust setup

```javascript
    import { Manager, LocalStoragePersistenceManager } from "matchmore";
    //...
    const localPersistenceManager = new LocalStoragePersistenceManager();
    await localPersistenceManager.load();
    this.manager = new Manager(
      apiKey,
      undefined,
      localPersistenceManager,
      // undefined,
      {
        enableHighAccuracy: false,
        timeout: 60000,
        maximumAge: 60000
      }
    )

```

{: #javascript-start-stop-location-updates}
#### Start/Stop location updates
```javascript
manager.startUpdatingLocation();
manager.stopUpdatingLocation();
```

{: #javascript-configure-custom-location-manager}
#### Configure custom location manager

{: #javascript-manual-update-locations}
##### Manual update locations

```javascript
await manager.updateLocation({
    latitude = 54.414662,
    longitude = 18.625498
});
await manager.updateLocation({
    latitude = 54.414662,
    longitude = 18.625498
}, deviceId);
```

{: #javascript-tutorials}
### Tutorials

{: #javascript-create-a-mobile-device}
#### Create a Mobile Device

First you need to create the main device

```javascript
//use your own application specific names
//device token can be kept empty, but it is used for third party match delivery mechanics which we will introduce to the sdk soon, it can be considered also optional
await manager.createMobileDevice("my mobile device", "iOS", "<device_token>");
manager.startMonitoringMatches();
manager.onMatch = (match) => console.log(match.publication.properties.name + " matched!");
```

{: #javascript-create-a-pin-device}
#### Create a Pin Device

```javascript
let newDevice = await manager.createPinDevice("pin", {
    latitude = 54.414662,
    longitude = 18.625498
});

```
{: #javascript-start-stop-monitoring-for-a-device}
#### Start/Stop Monitoring for a device
```javascript
manager.startMonitoringMatches();
```

{: #javascript-websocket}
#### WebSocket

```javascript
//explicitly pass the parameter to opt in for websockets
this.manager.startMonitoringMatches(MatchMonitorMode.websocket);
```

{: #javascript-polling}
#### Polling

```javascript
this.manager.startMonitoringMatches();
//or
this.manager.startMonitoringMatches(MatchMonitorMode.polling);
```

{: #javascript-publish}
#### Publish

```javascript
let publication =  await manager.createPublication(
      "my-topic",
      100 /* m */,
      20 /* s */,
      { "age": 20, "name": "Clara" });
```

{: #javascript-subscribe}
#### Subscribe

```javascript
let subscription = await this.manager.createSubscription(
      "my-topic",
      99999 /* m */,
      20 /* s */,
      "age >= 18"
    );
```

{: #javascript-get-matches}
#### Get Matches

```javascript
let matches = await manager.getAllMatches();
let otherDeviceMatches = await manager.getAllMatches(deviceId);
let specificMatch = await manager.getMatch(matchId);
```

{: #javascript-local-states-request}
#### Local States request

To access the local persisted entities you can just call them direct from the manager, they are regular array so you can query them whatever way you likes
```javascript
let devices = this.manager.devices;
let publications = this.manager.publications;
let subscriptions = this.manager.subscriptions;
```

{: #javascript-demo}
### Demo
Couple of demos to be found in [Javascript github](https://github.com/matchmore/js-sdk).

{: #javascript-changelog}
### Changelog

{: #javascript-supported-platform}
### Supported Platform
