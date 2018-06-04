---
title: Overview
sections:
  - Matchmore
  - In-depth Publish/Subscribe
  - Matchmore Cloud Service
  - Examples
  - Developer tools
---

## Matchmore
As the fast-growing world of mobile connected objects computing era is becoming a reality, Matchmore aims at becoming the leading cloud-based software platform supporting the creation of highly dynamic proximity-based applications. At the heart of our vision is the notion of geomatching, which is built on the ocation-based Publish/Subscribe communication model.

#### Mission

Our mission is to provide tools that dramatically simplify and accelerate the development, testing and deployment of rich application scenarios based on **multiple moving and connected objects**, making such applications the new low hanging fruits of the mobile app industry.

Get familiar with the Matchmore products and explore their features:

[*&nbsp;*{: .fa .fa-exclamation-circle} Start directly with our quickstart ](#quickstart){: .btn .btn-green .btn-cta}
[*&nbsp;*{: .fa .fa-exclamation-circle} How it works                       ](#how-it-works){: .btn .btn-blue .btn-cta}
[*&nbsp;*{: .fa .fa-exclamation-circle} Discover our SDKs                   ](#sdks-integration-configuration){: .btn .btn-orange .btn-cta}

### Understanding Matchmore

Matchmore helps you to model any geolocated or proximity-based applications, by taking advantage of advanced location-based Publish/Subscribe model. Create any type of interactions for connected objects (Internet of Things) from Smartphones to iBeacons and everything in between.

{: #in-depth-publish-subscribe}
#### In-depth Publish/Subscribe

The Publish/Subscribe model is a message-oriented paradigm in which, publishers can publish messages anonymously on a topic and subscribers can receive messages on that topic. The additional layer, namely selector, allows subscribers to filter incoming messages based on the message’s content.

{: #matchmore-cloud-service}
#### Matchmore Cloud Service

Matchmore is designed to simplify software developer’s life. It is an extension of the Publish/Subscribe model adding user context while remaining message-oriented. Publications and subscriptions are extended with the notion of geographical zone. The zone is defined as a circle with a center at a given location and a range around that location.

Publications and subscriptions are associated with any mobile device (i.e. smartphones and tablets), beacons or pin points and stay linked to it if the device moves. Both the crossing of the range and the correspondance of topic + content lead to the delivery of the message, called, a **match**.

To ease mobile application development, Matchmore is provided as various Software Development Kits (SDK) for different platforms.

#### Examples

Please have a look at some applications that have been built with Matchmore:

* [The steamLink](https://itunes.apple.com/us/app/the-steamlink/id1341462059?l=fr&ls=1&mt=8), built in less than 48 hours during the [Global Game Jam in Geneva](https://globalgamejam.org).

* [iTicketing Github Repo](https://github.com/matchmore/ios-ticketing-app)

* **Geomatching (Matching Moving Content)**

Matchmore enables you to notify your Application's users whenever there is someone or something they could be interested in close by. Create proximity detection easily on web or mobile by using the advanced location-based Publish/Subscribe paradigm.

* **Create Smart Geofences**

Geofences are used to define virtual areas in the real world, that you are particularly interested in.
With smart geofences, you can go further in the filtering process. Thanks to the topic and content filtering it can be used for analytics, for instance to count how many users enter a specific area based on their preference. Or for triggering content displaying matching on your Application. Matchmore works indoor (beacons) and outdoor (GPS).

* **Smart activation with IoT**

Activate a connected device when you approach it (for instance a camera).

* **Geochat**

Broadcast anonymous message to people around you.

* **Smart checkpoints**

Create virtual gates or points to control the passage of users. Example : A trail is validated if the user has passed by every point related to the trail.

<div class="callout-block callout-success"><div class="icon-holder">*&nbsp;*{: .fa .fa-thumbs-up}
</div><div class="content">
{: .callout-title}
#### Interesting, isn't it ?

You can add any of those features listed above with [geomatching](#geomatching). Simply use Matchmore to add location context and tracking to your apps with just a few lines of code.

Matchmore is applicable to many usecases and eases the modelization and deployment of any geolocated applications.

Want to get started quickly? Follow our [quickstart guide](#quickstart).

</div></div>

### Developer tools
You can integrate Matchmore in your apps using our developer tools: SDKs and the REST API.

#### SDK
Integrate the SDK into your iOS or Android applications to start generating matches. The SDKs abstracts away cross-platform differences between location services, allowing you to add location context and proximity detection to your apps with just a few lines of code.

Learn more about the SDKs.

<div class="row">
 <div class="col-md-6 col-sm-6 col-xs-12">

[*&nbsp;*{: .fa .fa-download}           iOS         ](#ios){: .btn .btn-orange .btn-cta}

 </div>
 <div class="col-md-6 col-sm-6 col-xs-12">

[*&nbsp;*{: .fa .fa-download}           Android     ](#android){: .btn .btn-orange .btn-cta}

 </div>
 <div class="col-md-6 col-sm-6 col-xs-12">

[*&nbsp;*{: .fa .fa-download}           Unity 3D    ](#unity-3d){: .btn .btn-orange .btn-cta}

 </div>
 <div class="col-md-6 col-sm-6 col-xs-12">

[*&nbsp;*{: .fa .fa-download}           JavaScript  ](#javascript){: .btn .btn-orange .btn-cta}

 </div>
</div>

#### API
You can manage your Matchmore objects, including devices, publications, subscriptions, and matches, using our restful API. For instance, you may use the API to list matches related to a device, or to create some publications programmatically.

Learn more about the [*&nbsp;*{: .fa .fa-download}           API ](https://matchmore.io/documentation/api){: .btn .btn-orange .btn-cta}

<div class="callout-block callout-info"><div class="icon-holder">*&nbsp;*{: .fa .fa-info-circle}
</div><div class="content">
{: .callout-title}
#### Questions ?

We're always happy to help with code or other questions you might have! Search our documentation, contact our support, or connect with our sales team.

</div></div>
