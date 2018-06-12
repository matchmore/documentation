---
title: JavaScript & React Native SDK
---

{: #javascript}
### Getting started

In a project backed by npm(like react) you can just type

```
npm install @matchmore/matchmore --save
```
or
```
yarn add @matchmore/matchmore
```

Or in a html document

```html
<script src="../../dist/web/matchmore.js"></script>
```


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

React.Native persistence requires some setup, best if put in its own file and import at the start of the application

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

Web setup can done inline like this

```javascript
matchmore.PlatformConfig.storage = {
  save: (key, value) => window.localStorage.setItem(key, value),
  load: (key) => window.localStorage.getItem(key),
  remove: (key) => window.localStorage.removeItem(key),
}
```

Developer is free to mix and match the styles

### Quickstart example

This is an example for web usage of the matchmore sdk

```javascript
 matchmore.PlatformConfig.storage = {
              save: (key, value) => window.localStorage.setItem(key, value),
              load: (key) => window.localStorage.getItem(key),
              remove: (key) => window.localStorage.removeItem(key),
            }
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

### Main device

First you need to create the main device

```javascript
//use your own application specific names
//device token can be kept empty, but it is used for third party match delivery mechanics which we will introduce to the sdk soon, it can be considered also optional
await manager.createMobileDevice("me", "iOS", "<device_token>");
manager.startMonitoringMatches();
manager.startUpdatingLocation();
manager.onMatch = (match) => {
                    console.log(match.publication.properties.name + " matched!");
                };
```


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

#### Create a Mobile Device

```javascript
let newDevice = await manager.createMobileDevice("another mobile", "android");
```

#### Create a Pin Device

```javascript
let newDevice = await manager.createPinDevice("pin", {
    latitude = 54.414662,
    longitude = 18.625498
});

```

#### Publish

```javascript
let publication =  await manager.createPublication(
      "my-topic",
      100 /* m */,
      20 /* s */,
      { "age": 20, "name": "Clara" });
```

#### Subscribe

```javascript
let subscription = await this.manager.createSubscription(
      "my-topic",
      99999 /* m */,
      20 /* s */,
      "age >= 18"
    );
```

#### GetMatches

```javascript
let matches = await manager.getAllMatches();
let otherDeviceMatches = await manager.getAllMatches(deviceId);
let specificMatch = await manager.getMatch(matchId);
```

#### Local persistence

To access the local persisted entities you can just call them direct from the manager, they are regular array so you can query them whatever way you likes
```javascript
let devices = this.manager.devices;
let publications = this.manager.publications;
let subscriptions = this.manager.subscriptions;
```

### DEMO
Couple of demos to be found in [Javascript github](https://github.com/matchmore/js-sdk).

### Changelog
### Supported Platform


