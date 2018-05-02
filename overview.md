---
title: Overview
sections:
  - Matchmore
  - In-depth Publish/subscribe
  - ALPS
  - Developer tools
---

## Matchmore
As the fast-growing world of mobile connected objects computing era is becoming a reality, Matchmore aims at becoming the leading cloud-based software platform supporting the creation of highly dynamic proximity-based applications. At the heart of our vision is the notion of geomatching, which builds on the location-based publish/subscribe communication model.

#### Description

Our goal is to provide tools that dramatically simplify and accelerate the development, testing and deployment of rich application scenarios based on **multiple moving and connected objects**, making such applications the new low hanging fruits of the mobile app industry.

Get familiar with the Matchmore products and explore their features:

**links to our products opens quickstart of || Matchmore SDK || Dashboard || Account**

### Understanding Matchmore
Matchmore helps you model any geolocated or proximity-based applications, by taking advantage of advanced location-based publish/subscribe (ALPS) model, create any type of interactions for connected objects from smartphones to any sensors and everything in between.

{: #in-depth-publish-subscribe}
#### In-depth Publish/subscribe
The purpose of Publish/Subscribe model is to create communication between several computers, applications, and users. Entities interested in a specific topic subscribe to the topic and whenever there is a message on this topic, they will receive it. Content-based filtering is an additional layer, namely selector, allowing subscribers to filter incoming messages based on the message’s content. On the other hand, publishers/senders are able to send messages without any knowledge of the future receivers, ensuring anonymity. Concerning content-based filtering, publishers are able to customize their messages with the additional layer, namely properties. For instance, a publication and a subscription were issued on the same topic and have compatible properties and the evaluation of the selector against those properties returns a true value, so there is a content correspondence which leads to a message delivery.
**/OR SHORTER/**

The publish/subscribe model is a message-oriented paradigm in which, publishers can publish anonymously on a topic and subscribers can receive messages on that topic. The additional layer, namely selector, allows subscribers to filter incoming messages based on the message’s content.

{: #alps}
#### Advanced Location based Publish/Subscribe
ALPS is designed to simplify software developer’s life. It is an extension of the Publish-Subscribe model adding user context while remaining message-oriented. Publications and subscriptions are extended with the notion of geographical zone. The zone is defined as a circle with a center at the given location and a range around that location.
Publications and subscriptions which are associated with a mobile device, e.g. user’s mobile phone, potentially follow the movements of the user carrying the device and therefore change their associated location. The crossing of the range and corresponding topic and content lead to the delivery of the message, in fact, a **match**.

To facilitate the mobile application development, ALPS is provided in enhanced Software Development Kits (SDK).

So what exactly can you do with Matchmore?

* **Geomatching (Matching Moving Content)**

ALPS enables you to notify the user whenever there’s someone/something they could be interested in close by. Create proximity detection easily on web or mobile by using the advanced location-based publish/subscribe paradigm.

* **Create Smart Geofences**

Geofences are used to define special areas in the real world, that you are particularly interested in.
With smart geofences, you can go further in the filtering process. It can be used for analytics uses: count how many app users based on their preference enter a specific area, thanks to the topic and content filtering. Or for triggering content inside the mobile or externally depending on user's attached pieces of information. Works indoor (beacons) and outdoor (GPS).

* **Smart activation with IoT**

Activate a connected device when you approach it (a camera by example).

* **Geochat**

Broadcast anonymous message to people around you.

* **Smart checkpoints**

Create virtual gates or points to control the passage of users. A trail is validated if the user has passed by every point related to the trail.

<div class="callout-block callout-success"><div class="icon-holder">*&nbsp;*{: .fa .fa-thumbs-up}
</div><div class="content">
{: .callout-title}
#### Interesting, isn't it ?

You can add any of those above listed features with [Proximity Detection](#proximity-detection). Simply use Matchmore to add location context and tracking to your apps with just a few lines of code.

We are convinced that Matchmore is applicable to many domains and eases the modelization of any geolocated applications.

Want to get started quickly? Follow our [quickstart guide](#quickstart).

</div></div>

<div class="callout-block callout-info"><div class="icon-holder">*&nbsp;*{: .fa .fa-info-circle}
</div><div class="content">
{: .callout-title}
#### Not convinced yet ? You can have a look at apps we have built with Matchmore

The steamLink, built in less than 48 hours during Game Jam Geneva.

iTicketing app.

</div></div>

### Developer tools
You can integrate Matchmore with your apps using our developer tools: the SDKs and the API.

#### SDK
Integrate the SDK into your iOS and Android apps to start tracking users and generating matches. The SDKs abstracts away cross-platform differences between location services on iOS and Android, allowing you to add location context and proximity detection to your apps with just a few lines of code.

Learn more about the SDKs.

[iOS](#ios) | [android](#android) | [unity 3D](#unity-3d) | [JavaScript](#javascript)

#### API
You can access all of your Matchmore data, including devices, publications, subscriptions, and matches, via the API. You might use the API to list matches related to a device, or to create publications programmatically.

Learn more about the [API](https://matchmore.io/documentation/api).

### What is next ? // (Examples of links : contact us, examples, tutorials)
### Recently added // (What next, OR recently added)
### Questions ? // (Should be here in all the pages)
We're always happy to help with code or other questions you might have! Search our documentation, contact support, or connect with our sales team.

[*&nbsp;*{: .fa .fa-download} Download PrettyDocs](http://themes.3rdwavemedia.com){: .btn .btn-green}
