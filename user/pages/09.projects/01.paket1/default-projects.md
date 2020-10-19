---
title: 'Travel Companion'
media_order: cover.JPG
routable: false
---

Travel Companion is an iOS app I developped from 2018 - 2019 inspired by my passion for travelling, which is available on the Apple Store.

Travel Companion's goal is to let explore, plan and remember travels. It has an integrated sign-in mechanism with either Facebook, Google, Apple, or email. This allows data synchronization over all your different devices. Here's a short demo video of the app:

<iframe width="375" height="500" style="display:block; margin: 0 auto;" src="https://www.youtube.com/embed/IEebsEjLKN8" frameborder="0" allowfullscreen></iframe>
<br>

Within the app CoreData and Firebase Cloud Firestore are used for data persistence as well as various API's (Google Maps, Google Places, Google Geocoding API, Firebase Authentication, Firebase Storage, Firebase Analytics, Firebase Crashlytics, flickR API, Rome2Rio and restcountries.eu).

Explore: The explore feature offers a Google Map that allows to search for desired destinations. Basic information about the destination and country as well as photos about the country, place and geographical location can be retrieved. Information provided by Wikipedia, Wikivoyage, Google and LonelyPlanet about the destination are offered, too.

Plan: The plan feature allows to add travel plans for destinations that you have previously explored. The travel plans include flights, public transport (including ferries), hotels, restaurants and attractions.

Flight and public transport search allows to enter an origin and destination with autocompletion support and a travel date. Available transport opportunities with detailed information are displayed in a TableView and can be added to the trip.

Hotels, restaurants and attractions can be searched by destination: You can drop a pin on the map, which triggers a search for the chosen place type and distance. The search can further be restricted by text search. Attractions are subdivided in various groups offered by Google Places, that can be chosen in a picker (e.g. amusement parks, zoos, general points of interest, etc.).

The plan overview displays all flights, public transports, hotels, restaurants and attractions in a grouped TableView. By tapping an item it is possible to delete it or to add a note. Furthermore, an image from the linked explore photos can be chosen by tapping the placeholder image.

Finally, plans are subdivided into upcoming and past trips, depending on the travel date.

Remember: The remember feature displays past trips (or trips that have been started), where you can add photos from the gallery or camera to remember your trip.

Details can be checked on Travel Companion's [homepage](https://travel-companion.firebaseapp.com/).

Of course, I'd be pleased about downloads and reviews on the [App Store](https://apps.apple.com/us/app/the-travel-companion/id1441554384).