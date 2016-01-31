---
author: Spencer Boucher
date: 2014-01-08
layout: post
summary: Let's put on our quantified-self hats and find out where we spend our time!
title: Live mapping
---

I've been wanting to do some more mapping stuff since my first encounter with
Leaflet a month or two ago while I was working on a project for [AutoGrid][]. I
had my eye on CartoDB's time series library, [Torque][], because I had really
wanted to do some time-series visualization, but time constraints and privacy
issues with uploading data to CartoDB's servers prevented me from really
exploring. Since I had a few days of free time over winter break, I played
around with it a bit and came up with this: [spencerboucher.com/map][]. How'd I
do it?

First I needed some geographic data, so I turned to a source of data I've been
collected for almost a year - my own location. [OpenPaths][] is a mobile app
that records your location at regular time intervals. I opted for every 30
minutes at first, then upped it to every 15 minutes when I discovered that the
effect on battery life wasn't nearly as bad as I expected it to be. OpenPaths is
a project of [the R&D department at The New York Times][] and they [claim][]
that you are the only one with access to the collected data. Interestingly, you
can grant various [research programs][] access to your data at your own
discretion. Your data is conveniently downloadable as a csv, json, or kml file,
so I easily pulled my dataset of \~3,000 time points since
December 2012. Unfortunately, I made the switch from iPhone to Android around
April (well, that part is fortunate), and forgot to re-download the app, so I
only really have data from the around the first three months and last two months
of 2013.

Turns out, making impressive maps with CartoDB is almost embarrassingly
easy. Their GUI is pretty intuitive and running queries on their postgreSQL
database is simple. Even time series stuff built on the Torque backend is really
just point and click. I decided that the best way to visualize this data was
with an aggregated hexbin heatmap of all my past locations, overlaid with a
point-by-point replay with a time-slider. From there, it was just a one-line API
call to host the map on my website (line 30 highlighted below), which is
significantly easier than the legwork that went into crafting a Leaflet map
"manually."

```html
<html>

<head>

  <meta name="viewport" content="initial-scale=1.0, user-scalable=no" />

  <title>Location | Spencer</title>

  <link rel="shortcut icon" href="http://spencerboucher.com/map/favicon.png" />
  <link rel="stylesheet" href="http://libs.cartocdn.com/cartodb.js/v3/themes/css/cartodb.css" />
  <!--[if lte IE 8]>
    <link rel="stylesheet" href="http://libs.cartocdn.com/cartodb.js/v3/themes/css/cartodb.ie.css" />
  <![endif]-->

  <style type="text/css">
      html, body, #map {
          margin: 0;
          padding: 0;
          width: 100%;
          height: 100%;
          background: black; }
      #cartodb-gmaps-attribution {
        visibility: hidden; }
  </style>

  <script src="http://maps.google.com/maps/api/js?v=3.2&sensor=false"></script>
  <script src="http://libs.cartocdn.com/cartodb.js/v3/cartodb.js"></script>
  <script>
    function init(){
      cartodb.createVis('map', 'http://justmytwospence.cartodb.com/api/v2/viz/e8fd87d0-78b3-11e3-a9e9-e7941b6e2df0/viz.json'); }
  </script>

</head>

<body onload="init()">
  <div id='map'></div>
</body>

</html>
```

This is pretty awesome, but in light of how easy it all was, I was almost
disappointed. Can we take it one step further? Let's put on our
[Quantified Self][] hats and set about to make this map *live*. There's three
components to making this happen, so we'll step through them one at a
time. First we need to access the most recent data from OpenPaths (there's an
API for that!), and then we need to insert that data into CartoDB's database
(guess what, there's an API for that too). Last but not least, we need to
schedule that data transplant to occur on a regular basis. The Unix utility
`cron` is the canonical tool for this type of thing, so this seemed like a good
time to learn how to use it.

Python has a reputation for being a great "glue" language, so that's what we'll
use to build this script.

Programmatically accessing your data from OpenPaths is super simple. This piece
of our script is pulled more or less verbatim from
[the OpenPaths API documentation][]. Line 21 (highlighted below) is key - this
is where we specify which data you want to pull for injection into the CartoDB
database. Here we will grab the last 24 hours of data (\~96 readings, if you're
collecting every 15 minutes like me), getting the results in a nice
JSON-formatted variable named `data`.

```python
import oauth2, time, urllib, urllib2, json

ACCESS = 'redacted'
SECRET = 'redacted'
URL = 'https://openpaths.cc/api/1'

def build_auth_header(url, method):
    params = {
        'oauth_version': "1.0",
        'oauth_nonce': oauth2.generate_nonce(),
        'oauth_timestamp': int(time.time()),
    }
    consumer = oauth2.Consumer(key=ACCESS, secret=SECRET)
    params['oauth_consumer_key'] = consumer.key
    request = oauth2.Request(method=method, url=url, parameters=params)
    signature_method = oauth2.SignatureMethod_HMAC_SHA1()
    request.sign_request(signature_method, consumer, None)
    return request.to_header()

now = time.time()
params = {'start_time': now - 24*60*60, 'end_time': now} # get the last 24 hours
query = "%s?%s" % (URL, urllib.urlencode(params))

[AutoGrid]: https://auto-grid.com

[Torque]: https://github.com/cartodb/torque

[OpenPaths]: https://openpaths.cc

[the R&D department at The New York Times]: https://nytlabs.com
#print(query)
try:
    request = urllib2.Request(query)
    request.headers = build_auth_header(URL, 'GET')
    connection = urllib2.urlopen(request)
    data = json.loads(''.join(connection.readlines()))
    print(json.dumps(data, indent=4))
except urllib2.HTTPError as e:
    print(e.read())
```

Now we need to get our new `data` variable into CartoDB's postgreSQL
server. [CartoDB's SQL API documentation][] makes this possible, and there's
even a [python module][] that wraps OAuth2 to simplify things. Although its
still in the early stages of development, this module works fine for our current
purposes; all we have to do is send it a string that holds the SQL query we want
to run. So now we'll just write a for-loop that successively builds an `INSERT`
query for each element in `data` (lines 18-20 highlighted below).

```python
from cartodb import CartoDBException, CartoDBAPIKey

user =  'spencer.g.boucher@gmail.com'
password =  'redacted'
cartodb_domain = 'justmytwospence'
API_KEY ='redacted'
cl = CartoDBAPIKey(API_KEY, cartodb_domain)

for reading in data:
    alt     = str(reading['alt'])
    device  =     reading['device']
    lat     = str(reading['lat'])
    lon     = str(reading['lon'])
    os      =     reading['os']
    t       = str(reading['t'])
    version =     reading['version']
    try:
        query_string = "INSERT INTO openpaths_justmytwospence (alt, date, device, lat,  lon, os, version, the_geom) "
                       "VALUES ({0}, abstime({1}), '{2}', {3}, {4}, '{5}', '{6}', ST_ SetSRID(ST_Point({4}, {3}), 4326))"
                      .format(alt, t, device, lat, lon, os, version)
        print cl.sql(query_string)
    except CartoDBException as e:
        print ("some error ocurred", e)
```

A few notes:

-   It would certainly be faster to insert all of the new data into the
    database using a single `INSERT` statement, but that would require
    some more tedious text parsing and execution speed isn't
    particularly important to us. As it stands, it takes about six
    seconds to post a day's worth of data.
-   One posgreSQL "gotcha" had me hung up for quite some time: single quotes
    parse fine but double quotes do not.
-   `ST_SetSRID` is a [PostGIS command][] that converts a lon/lat pair
    (in that order - another "gotcha") to the necessary geometry object.

Last but not least, we need this script to run automatically. Because we've
written the script to transplant 24 hours of data, we'll need to run it once a
day in order to capture all of the data that's being generated. I tried to set
up my web host, [LaughingSquid][], to do this, but unfortunately they don't
grant shell access so we can't install all those fancy python modules that we've
already used. Its totally possible to rewrite the script to use only modules
from the [Python Standard Library][], but this would turn a simple task into a
tedious one. Manually implementing OAuth in particular would be a total pain in
the rear, and classes are just about to resume after all, so a different
solution is in order. Let's spin up a ["micro" EC2 instance][] instead. This
gives us free reign to install whatever we need for the low low cost of ¢.02 per
hour. This does start to add up, but our Master's program gives us some pretty
substantial Amazon Web Services credit that goes mostly unused, so we aren't too
upset :). UPDATE: A new post provides details about how to schedule Amazon EC2
instances - <http://www.spencerboucher.com/ec2-apis/>.

After `pip install`ing everything we need and `scp`ing our python script (let's
call it update.py) into the home directory of our remote server, all we need to
do is set up a crontab with the `crontab -e` command and add the following line:

```bash
@daily /usr/bin/python ~/update.py
```

`@daily` is actually a shortcut for `* * * * *`, where each asterix is a
placeholder for the (respectively) minute, hour, day of month, month, and day of
week that the script should executed. This shortcut defaults to midnight every
day, which is really as good as anything for our purposes.

Voilà! Now we can step back and relax, knowing that we don't have to do a single
thing and our map will continue to show the most up-to-date data available.

A few final notes:

 - We might reasonably want to lag our script by a week or so, for
   security/privacy reasons.
 - As far as I can tell, the location readings are recorded in a [POSIX time][]
   and have not been adjusted by time zone, so they are still in the [UTC][]
   time zone. This means that they are 8 hours off from the actual time in
   California, where I usually am. This doesn't bother me too much at the moment
   because the visualization is still relatively low resolution in the time
   domain anyways. At some point I might implement the relevant transformation,
   but this will raise its own issues because I won't *always* be in California,
   not to mention all that Daylight Savings nonsense.
 - [There is an addendum][] to this post that will take you through how to schedule the EC2 instance and avoid having it run 24/7.

  [spencerboucher.com/map]: http://www.spencerboucher.com/map
  [claim]: https://openpaths.cc/FAQ
  [research programs]: https://openpaths.cc/projects
  [Quantified Self]: http://quantifiedself.com/about/
  [the OpenPaths API documentation]: https://openpaths.cc/api
  [CartoDB's SQL API documentation]: http://developers.cartodb.com/documentation/sql-api.html
  [python module]: https://github.com/vizzuality/cartodb-python
  [PostGIS command]: http://postgis.org/docs/ST_SetSRID.html
  [LaughingSquid]: https://laughingsquid.us/
  [Python Standard Library]: http://docs.python.org/2/library/
  ["micro" EC2 instance]: http://aws.amazon.com/
  [POSIX time]: http://en.wikipedia.org/wiki/Unix_time
  [UTC]: http://en.wikipedia.org/wiki/Coordinated_Universal_Time
  [There is an addendum]: http://www.spencerboucher.com/ec2-apis/
