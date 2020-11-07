---
title: 'Travel Companion - An iOS app to find, organize and remember travels'
media_order: tc.jpg
date: '14:55 07/29/2018'
taxonomy:
    category:
        - blog
    tag:
        - iOS
        - TravelCompanion
theme: magnet-theme
feed:
    limit: 10
author: 'Stefan Jaindl'
hero_classes: 'text-light title-h1h2 overlay-dark-gradient hero-large parallax'
hero_image: tc.gif
show_sidebar: true
header_id: blog
badge_class: badge-color-blog
---

The first things I thought about before implementing the app have been:

1. What features should be built into the app?

2. How should it look like?

3. How should data persistence be handled?

4. Which APIs should be used?


Therefore, I want to elaborate a little bit on these points.

===


Generally, there many apps in the app store, that support finding information about travel destinations, organize travels or store photos in a categorized way. But I miss an app that supports all three things in one place. Therefore I've decided to create my own app with all these features.


In detail I want to offer:

Facebook, Google and E-Mail Login Authentication
I've already implemented such features for Pocket Code's Android and Web versions, therefore I already had knowledge on these topics. See https://www.catrobat.org for details on the Catrobat umbrella project.

Discover feature
A Google Maps or MapKit that allows to drop pins on the map and retrieve basic information on as well as photos about the destination.Users should be able to mark countries as "want", "been" and "lived", too. Finally, it should also support my favorite feature of Google Earth: Jump to a random travel destination.

Plan feature
The plan feature should allow to plan a trip including flight information, public transport, accomodations/hotels and activities. These points can either be searched for with usage of APIs or added manually. Furthermore, the travel route can be planned on a map. The entire travel plan can be shared. An overview of planned trips is shown in a TableView.

Remember feature
As soon as a planned trip is over, it should be possible to add special travel memories. Therefore, I want to add a photo gallery on a per-trip basis.


For the design, I've made a design prototype of the app using Adobe XD. I've recorded a video of the design:

<iframe width="375" height="500" style="display:block; margin: 0 auto;" src="https://www.youtube.com/embed/SmxQhBlhWOY" frameborder="0" allowfullscreen></iframe>
<br>



For data persistance I decided to mainly use Firebase Realtime Database, as it is well-tested and the database can be synched to Android (which isn't the case with CoreData), which is important when I decide to implement the app for Android, too. This encompasses countries, trips, flights, transports, hotels, attractions and the remember gallery.



The only thing I want to implement with CoreData are the discover-pins and photos, as they can easily be re-retrieved on other platforms.


As Web APIs I want to use well-tested and documented APIs that are steadily supported. After some research, I decided to use the following APIs:

* Google Maps (Map features)
* Google Geocoding API (convert address strings to latitude/longitude for search)
* Firebase Authentication (Google- & Facebook-SignIn, E-Mail-Login)
* Firebase Realtime DB (Online Database)
* Firebase Storage (Online photo storage)
* Firebase Analytics (Collect App Analytics Data to better understand users)
* Firebase Crashlytics (Retrieve crash reports)
* flickR API (retrieve Photos for a given latitude/longitude)
* Rome2Rio (transportation - Provides a simple XML/JSON interface for adding multi-modal search capability to your website, app or service. Returns train, bus, ferry, air, driving and walking routes between any two points. Inputs may be specified either as textual place names or latitude/longitude co-ordinates)
* Google Place API/Direction API (Planning routes)
* Tripexpert (Allows search for hotels, restaurants and attractions + Photos for planning trips)
* http://restcountries.eu (Country data for discover feature)

The [Quark theme](https://getgrav.org/downloads/themes) has the ability to allow pages to override some of the default options by letting the user set `body_classes` for any page.  The theme will merge the combination of the defaults with any `body_classes` set. This allows you to easily add hero classes to give your blog post some **bling**.

===

## Body Classes

```yaml
body_classes: "header-dark header-transparent"
```

On a particular page will ensure that page has those options enabled (assuming they are false by default).

## Hero Options

The hero template allows some options to be set in the page frontmatter. This is used by the modular `hero` as well as the blog and item templates to provide a more dynamic header.

```yaml
hero_classes: text-light title-h1h2 parallax overlay-dark-gradient hero-large
hero_image: road.jpg
hero_align: center
```

The `hero_classes` option allows a variety of hero classes to be set dynamically these include:

* `text-light` | `text-dark` - Controls if the text should be light or dark depending on the content
* `title-h1h2` - Enforced a close matched h1/h2 title pairing
* `parallax` - Enables a CSS-powered parallax effect
* `overlay-dark-gradient` - Displays a transparent gradient which further darkens the underlying image
* `overlay-light-gradient` - Displays a transparent gradient which further lightens the underlying image
* `overlay-dark` - Displays a solid transparent overlay which further darkens the underlying image
* `overlay-light` - Displays a solid transparent overlay which further darkens the underlying image
* `hero-fullscreen` | `hero-large` | `hero-medium` | `hero-small` | `hero-tiny` - Size of the hero block

The `hero_image` should point to an image file in the current page folder.


```
link: http://daringfireball.net
```
