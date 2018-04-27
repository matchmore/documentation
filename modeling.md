---
title: Modeling
sections:
  - Location
  - Device
  - Mobile
  - Pin
  - iBeacon
---

{: #modeling}
### Modeling
### Location and Device
TODO text qui parles de la liaison entre location et device


{: #location}
### Location
A location consists of a geographical position of a Device. A conventionally known *update location* is a `Location` object sent to Matchmore cloud service to inform of changes upon position for a device.

* createdAt
integer <int64>
The timestamp of the location creation in seconds since Jan 01 1970 (UTC).

* latitude
number <double> Required
The latitude of the device in degrees, for instance '46.5333' (Lausanne, Switzerland).

* longitude
number <double> Required
The longitude of the device in degrees, for instance '6.6667' (Lausanne, Switzerland).

* altitude
number <double> Required
The altitude of the device in meters, for instance '495.0' (Lausanne, Switzerland).

* horizontalAccuracy
number <double>
The horizontal accuracy of the location, measured on a scale from '0.0' to '1.0', '1.0' being the most accurate. If this value is not specified then the default value of '1.0' is used.

* verticalAccuracy
number <double>
The vertical accuracy of the location, measured on a scale from '0.0' to '1.0', '1.0' being the most accurate. If this value is not specified then the default value of '1.0' is used.


{: #device}
### Device
A device might be either virtual like a pin device or physical like a mobile device.
Devices may issue publications and subscriptions at any time; it may also cancel publications and subscriptions issued previously. Publications and subscriptions do have a definable, finite duration, after which they are deleted from the Matchmore cloud service and don’t participate anymore in the matching process.


#### Mobile
A mobile device is one that potentially moves together with its user and therefore has a geographical location associated with it. A mobile device is typically a location-aware smartphone, which knows its location thanks to GPS or to some other means like cell tower triangulation, etc.

* id
string
The id (UUID) of the device.

* createdAt
integer <int64>
The timestamp of the device's creation in seconds since Jan 01 1970 (UTC).

* updatedAt
integer <int64>
The timestamp of the device's creation in seconds since Jan 01 1970 (UTC).

* name
string
The name of the device.

* platform
string Required
The platform of the device, this can be any string representing the platform type, for instance 'iOS'.

* deviceToken
string Required
The deviceToken is the device push notification token given to this device by the OS, either iOS or Android for identifying the device with push notification services.

* location
Location Required

#### Pin
A pin device has an ultimate geographical location associated with it but is not represented by any object in the physical world; usually it’s location doesn’t change frequently if at all.

* id
string
The id (UUID) of the device.

* createdAt
integer <int64>
The timestamp of the device's creation in seconds since Jan 01 1970 (UTC).

* updatedAt
integer <int64>
The timestamp of the device's creation in seconds since Jan 01 1970 (UTC).

* name
string
The name of the device.

* location
Location Required


##### Using Pin Device

#### iBeacon
Beacons are high-tech tools that repeatedly broadcast a single signal under the form of advertising packet. Other devices interact with beacons through bluetooth and receive an advertising packet which consist of different letters and numbers. With the information received through the packet, devices like smartphones know how close they are to a specific beacon. The main purpose behind beacons is to improve indoor location. When developers know how close they are to this specific location, thanks to beacons, they can do something useful with this information.

* name
string
The name of the device.

* proximityUUID
string Required
The UUID of the beacon, the purpose is to distinguish iBeacons in your network, from all other beacons in networks outside your control.

* major
integer <int32> [ 0 .. 65535 ] Required
Major values are intended to identify and distinguish a group.

* minor
integer <int32> [ 0 .. 65535 ] Required
Minor values are intended to identify and distinguish an individual.


##### Beacons standard
The iBeacon is the standard defined by Apple. There is also other beacons standard like Eddystone by Google or AltBeacons by Kontakt.io. Basically, the content of the advertising packet could vary slightly from one standard to another, but the communication protocol (bluetooth) remains the same. As a consequence, most beacons on the market support at least the iBeacon standard and the Eddystone standard.

{: #ios}
