
### Tutorials
In the following section, discover how to use Matchmore with some great code examples.
#### Create a Mobile Device
The mobile device is the running app device. Generally, you have only one mobile device per user.

```JSON
curl -X POST \
  http://api.matchmore.io/v5/devices \
  -H 'api-key: f3823829-b738-4897-a971-2c64cfa09915' \
  -H 'content-type: application/json' \
  -d '{
  "group": [
	"B",
	"A"
  ],
  "name": "My iphone",
  "deviceType": "MobileDevice",
  "platform": "iOs 11",
  "deviceToken": "some-apn-token-for-push",
  "location": {
    "latitude": 0,
    "longitude": 0,
    "altitude": 0
  }
}'
```
#### Create a Pin Device
The pin device is a virtual non-physical geolocation described by latitude and longitude.
Pin devices are used to represent non-moving object in your world. Imagine a store, you can represent it by creating a pin device.

```JSON
curl -X POST \
  http://api.matchmore.io/v5/devices \
  -H 'api-key: f3823829-b738-4897-a971-2c64cfa09915' \
  -H 'content-type: application/json' \
  -d '{
  "deviceType":"PinDevice",
  "name":"Pin 1",
  "group": [
      "A",
      "cc"
  ],
  "location":{
      "latitude":0.0,
      "longitude":0.0,
      "altitude":0.0
  }
}'
```
#### Create a Beacon Device
The beacon device is the virtual representation of your beacons conformed iBeacons.
Beacons are helping indoor location based services.

The process to create beacon device is slightly different to other devices.

Connect to Matchmore portal with your account and click on "Beacons" tab.

Click on "Register a new Beacon" button at the top-right of the page.
Fill in with your beacon's information (UUID, Major and Minor).

Your beacon is now registered and can be used in your Matchmore app.

Go to "Apps" tab, and find the application in which you want to use your beacon(s).

Click on the "+" at the bottom of an application's information, and tick the beacon(s) you want to assign to this app.

Your beacons are now ready to be used in the application.
#### Publishing
When attached to a device, a publication is a message that is broadcasting the attached device presence to nearby subscribers.

```JSON
curl -X POST \
  http://api.matchmore.io/v5/devices \
  -H 'api-key: f3823829-b738-4897-a971-2c64cfa09915' \
  -H 'content-type: application/json' \
  -d '{
  "deviceId": "78378b1a-d64d-4b88-beec-65294c6d269e",
  "worldId": "f3823829-b738-4897-a971-2c64cfa09915",
  "topic": "my topic",
  "range": 10,
  "duration": 100,
  "properties": {
    "age": "32",
    "naminn": "pub_42"
  }
}'
```
#### Subscribing
When a device is interested to find nearby devices, it starts to subscribe.

```JSON
curl -X POST \
  http://api.matchmore.io/v5/devices \
  -H 'api-key: f3823829-b738-4897-a971-2c64cfa09915' \
  -H 'content-type: application/json' \
  -d '{
  "worldId": "13823829-b738-4897-a971-2c64cfa09915",
  "deviceId": "18378b1a-d64d-4b88-beec-65294c6d269e",
  "topic": "my topic",
  "selector": "age>30",
  "range": 20,
  "duration": 100,
  "pushers": [
    "localhost"
  ]
}'
```
#### GetMatches
