---
author: Spencer Boucher
date: 2014-12-01
layout: post
summary:
title: Maptime at Stamen
---

Last week, a classmate and I took a break from coursework to attend one of the
many great Meetup events that San Francisco has to offer for data science
practitioners. I’ve been pushing myself to attend at least one data-centric
Meetup every week, because these events are one of the most amazing parts about
going to school in the same place where so many of the biggest names work. To be
honest, I believe that becoming a presence in the data science scene and meeting
the movers and shakers is equally if not more important than coursework.

I actually attended 3 meetups last week, one about D3.js at Trulia HQ, one about
GIS technologies and the Code for America HQ, and one about mapping at Stamen
HQ. I picked all three because they are relevant to a geospatial data
visualization that I am working on for my practicum at AutoGrid, but the last
one is what I’m going to talk a bit about, because it was the most hands-on.

The workshop took place at Stamen Design’s headquarters in the Mission and was
led by Eric Theise; you can see his beautiful/informative slides (created using
[reveal.js][]) [here][]. Some useful Q&A happened on
[the Meetup event page as well][].

This was actually a two-part workshop, but it was relatively painless to follow
the instructions and get up to speed for the part II, so you should really give
it a shot even now if it looks interesting.

First we got postgres up and running on our machines. I have local installations
of MySQL, MongoDB, Hadoop and Hive up and running thanks to our course in
Distributed Databases, but our class didn’t have time to get to postgres within
a single credit hour. This, despite the fact that our professor admits to
postgres being the best database to use if you have anything to say about it.

Next, we populated our database with some data from OpenStreetMap. Mike Migurski
extracts data from OSM for major metropolitan areas on a semi-regular basis, so
we used the San Francisco data [available on his web site][] via [osm2pgsql][],
a command-line utility that loads OpenStreetMap data into PostgreSQL databases.

Then we used [TileMill][], MapBox’s desktop application, to visualize our
newborn database. We discovered how remarkably easy it can be to create vector
layers for data contained in such a postGIS database using the same old SQL and
CSS syntax you already know and love. Eric introduced us to some sensible
pre-baked [CartoCSS boilerplate][] courtesy of Andy Allen.

Lastly, we used a nifty feature of TileMill to actually bake our own map tiles
and serve them up for use in our own maps. Note that if you want to do this,
you’ll need the [mbutil command-line utility][], not currently mentioned in the
slide deck.

Not too shabby for 2 hours on a Wednesday night. Many thanks to the guys at
Stamen for hosting, especially Eric for all his work on the slides. Not to
mention the many other brilliant people who have made the tools and resources
that allow something this involved and grandiose to be done on a laptop by
someone who is still learning the ropes. Hands-on workshops like this are one of
the best ways to learn these technologies. Case in point, I may not have ever
stumbled across Mike or Andy’s resources had I not been learning directly from
people who are intimately familiar with the practical ins and outs of digital
cartography.

Digital mapping is rapidly capturing my interest because of the beautifully
functional things one can do with it, and it seems like an amazing time to be
learning it, because the ecosystem is beginning to really flourish. Looking
forward to more events!

[reveal.js]: http://lab.hakim.se/reveal-js

[here]: http://erictheise.github.io/geostack-deck

[the Meetup event page as well]: http://www.meetup.com/Maptime-SF/events/147110652

[osm2pgsql]: http://wiki.openstreetmap.org/wiki/Osm2pgsql

[TileMill]: https://www.mapbox.com/tilemill

[CartoCSS boilerplate]: https://github.com/gravitystorm/openstreetmap-carto

[mbutil command-line utility]: https://github.com/mapbox/mbutil
