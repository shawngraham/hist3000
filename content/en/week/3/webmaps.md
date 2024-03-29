---
title: "Webmaps"
description: "Webmapping with Leaflet"
date: 2020-01-28T00:10:51+09:00
draft: false
weight: -7
author: "Shawn Graham"
---

### Introduction

In this tutorial, I re-purpose some instructions from my Crafting Digital History class on how to build a web map using open source code from Leaflet. The idea here is that you learn to build a map and get it online: and then you modify it to display

- the cemetery that you have worked on
- the location of stones

The `East End Cemetery` website maps a historic African American burial ground in Henrico County and the city of Richmond, Virginia, and uses Leaflet:

[East End Cemetery Map](https://dsl.richmond.edu/eastend/#loc=16/37.53619777622634/-77.38787770271303)

With time and practice, you would be able to build something similar. Remember, I am not expecting perfection; rather, I want you to push yourself until you get stuck.

{{< alert theme="info" >}}
**Nb** When I talk about 'serving' your site in step 8, I am assuming you have python installed on your computer. If you have a mac, you probably do; if you have a PC, probably not. If you don't want to muck about with Python, you can download the free trial of [prepos](https://prepros.io/), which will act like a web server. Open that program, and then drag and drop the folder you are working in (always start a new project like this in a new folder) onto the prepos interface. [See these instructions](https://prepros.io/help/getting-started).
{{</alert >}}

By the way, subsurface georadar and other non-invasive techniques are a part of digital archaeology that can transform how we understand historical graveyards - take a look at [this twitter thread](https://twitter.com/ben_hitchcock_/status/1298617256914624517).


### Webmapping with Leaflet

> The following is a modified exercise from the [Open Digital Archaeology Textbook Environment](https://o-date.github.io/draft/book/what-is-web-mapping.html) by Graham et al 2019; this particular section was contributed by [Neha Gupta of UBC](https://dngupta.github.io/).

First of all, a webmap is not a Geographic Information System or GIS. A webmap may be _derived_ from a GIS, but the key difference is that a GIS allows analytical questions to be asked of, and derived from, different _layers_ of spatial or spatialized data. Webmaps are used more for conveying the results of such analyses.

Tiled map service and [Web Map Service](http://www.opengeospatial.org/standards/wms) are two forms of Web-based mapping. A WMS is an interface that enables us (the clients) to request specific maps i.e. visual representations of geographic information from a geospatial database. The WMS server is called via a Universal Resource Link (URL) on an Internet-enabled desktop GIS. A request typically consists of the geographic layer (e.g. theme) and geographic area of interest. The response to a request results in geo-registered map images that are displayed and queried within a browser. Because the map is dynamically drawn upon request, and because the server typically uses the most current information from several layers in the geospatial database, WMS maps tend to load slowly.

Tile map services such as [TileMill Project](https://tilemill-project.github.io/tilemill/docs/crashcourse/introduction/), [OpenStreetMap](https://www.openstreetmap.org/), and [Stamen](http://maps.stamen.com/) all use one or more vector layers that have been rasterized into an image. This rasterized image is divided into 256 x 256 adjacent pixel images or ‘tiles’. This is usually the base layer in a web map.

Each tile is an image on the Web, which means that you can link to it. For example, the following URL points to a specific tile on the Web:

```
https://tile.openstreetmap.org/7/63/42.png
```

The three elements in the URL are:

1. tile.openstreetmap.org, the tile server name;
2. 7, the zoom level or z value of the tile; and finally
3. 63/42, the x and y values in the grid where the tile lives

The z value has a range between 0 and 21, where 21 returns a tile with greatest detail (and smallest sized tile).

![](https://o-date.github.io/draft/book/images/tile-pyramid.jpg)

'Image Tile Pyramid, [https://www.azavea.com/blog/2018/08/06/generating-pyramided-tiles-from-a-geotiff-using-geotrellis/tilepyramid/](https://www.azavea.com/blog/2018/08/06/generating-pyramided-tiles-from-a-geotiff-using-geotrellis/tilepyramid/)’

Once generated, the set of tiles are stored on disk, ready to be distributed rapidly to large numbers of simultaneous requests. Tiled maps load quickly precisely because they are pre-generated. They shift attention to map aesthetics and smooth map navigation, trading in functionality such as layer order, map scale and projection. [Alex Urquhart](http://alexurquhart.github.io/free-tiles/) maintains a list of tile services.

Data layers are typically added on top of the base layer. Data layers can be points, lines and polygons. These data layers are saved as [GeoJSON](https://geojson.org/), a format designed for representing on the Web, geographic features with their non-spatial attributes.

[Leaflet](http://leafletjs.com/) is a JavaScript library developed by [Vladimir Agafonkin](https://vimeo.com/106112939) for use with tiled maps. Launched in 2008, Leaflet has become widely used in tile web mapping because the library’s low-barrier customization and interactivity with map elements, and because of its simplified setup when compared to a WMS served map. Moreover, Leaflet’s compatibility with other Web 2.0 technologies and code-sharing platforms such as GitHub has encouraged an active community of ‘makers’.

You will need a proper text editor (not notepad, not, *shudder*, Word) to edit your files. I suggest you download, install, and use [Sublime Text](https://www.sublimetext.com/).

1. Create a new directory on your machine and call it 'web-map'
2. Open a terminal or command prompt in that directory.
3. We're going to build a map now; in sublime text make a new file and save it as `index.html` in the `web-map` folder.
4. We'll begin by putting some important information about the building blocks we'll use in the 'header' of our file, between the `<head>` and `<\head>` tags. Copy and paste the following into your index.html:

```
<head>

	<title>A Demo WebMap</title>

	<meta charset="utf-8" />
	<meta name="viewport" content="width=device-width, initial-scale=1.0">

	<link rel="stylesheet" href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css" />
	<script src="https://unpkg.com/leaflet@1.6.0/dist/leaflet.js"></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>
    <script type="text/javascript"></script>
    </script>

</head>
```
This is telling the browser the name of this page, and it's telling the browser where to get the stylesheet and the code for making the map work. You don't have the `leaflet.css` or the `leaflet.js` files yet. Right-click, and 'save as' copies of these two files into your web-map folder from here: [leaflet.css](/data/leaflet.css) and [leaflet.js](/data/leaflet.js)

5. The next bit of code defines the base map:

```
<body>
<div id="mapid" style="width: 600px; height: 400px;"></div>
<script type="text/javascript">
var myMap = L.map('mapid').setView([45.4192857, -75.6973237],13);
L.tileLayer('https://stamen-tiles-{s}.a.ssl.fastly.net/toner/{z}/{x}/{y}.png',
	{maxZoom: 19
}).addTo(myMap);

```

We're creating a box on the page called 'mapid' and setting its width and height in pixels. Then our script creates a thing called 'mymap' and defines some particulars about it: the center of the map (here, Ottawa, Ontario), the zoom level, and the basemap layer. The final bit, `addTo(mymap);` does the work of putting all of that information on the map (using leaflet.js).

6. Now we're going to need some data for our map. Our point data will be encoded in a format called 'geojson', which is a kind of list. If you have geographic data in a csv table, with latitude and longitude columns, you could use [this tool](http://www.convertcsv.com/csv-to-geojson.htm) to convert it to geojson. For now, our geojson has two points in it:

```
{
   "type": "FeatureCollection",
   "features": [
  {
    "type": "Feature",
    "geometry": {
       "type": "Point",
       "coordinates":  [-75.6973237, 45.4192857]
    },
    "properties": {
    "Label":"Centre of our map!"
    }
  },
  {
    "type": "Feature",
    "geometry": {
       "type": "Point",
       "coordinates":  [ -75.699222, 45.4275 ]
    },
    "properties": {
    "Label":"Not the centre of the map "
    }
  }
]
}

```

Copy that into a new file and save in your web-map folder as `point-data.geojson` . (When you try to build a map of your graveyard, do you see how you could add your own point data with the csv-to-geojson tool?)

7. Now append the following to the `index.html`, which will tell leaflet where the data is and how to add it to our map:

```
// load a GeoJSON from external file
$.getJSON("point-data.geojson", function(data) {
  // add GeoJSON layer to the map once the file is loaded
  var datalayer = L.geoJson(data ,{
  onEachFeature: function(feature, featureLayer) {
  featureLayer.bindPopup(feature.properties.Label);
  }
}).addTo(myMap);
});

</script>

</body>
</html>
```

Save!

8. At the command prompt, you'll use python to start a webserver; the webserver acts like a regular server and makes these two files understandable by your web-browser: `$ python -m http.server` **Please see the alert at the top of the page if you don't have python on your machine; this is where you drag your work folder into prepos**.

9. The terminal will tell you now that your server is running at an address that will look something like this: `(http://0.0.0.0:8000/)`. Copy that full address and paste it into your browser's address bar. (It might look like `http://127.0.0.1:8000` or `http://localhost:8000`)

Congratulations! You have made a webmap! Do you see how you could recenter the map on Ottawa? Add some more data? Change the base map to another style?

(You can upload your web-map folder to Github by dragging the folder onto a repository; to make Github _serve_ your website, you create a new branch called `gh-pages`. So if I loaded it into github.com/shawngraham/demo/webmap in the gh-pages branch, my map will be available at shawngraham.github.io/demo/webmap/index.html. Cool, eh?)

### Adding a new layer to our map

[Map Warper](https://mapwarper.net) is a service that allows us to upload images of maps, and then tie 'control points' to real-world geography such that we stretch and warp the image to overlay the physical world. It then provides us with a tile-server so that the image can be imported into our map as its own layer. Take a look at this map of Ottawa:

![Ottawa 1894](https://legacy.lib.utexas.edu/maps/historical/ottawa_1894.jpg)

This was uploaded to Mapwarper and georectified by a former MA student at Carleton. You can see it on Mapwarper [at this location](https://mapwarper.net/maps/29435). (*nb* Mapwarper periodically runs out of space; so look for maps that were last warper/used within the last two years ago. For maps that were last updated longer than two years ago you might need to log into Mapwarper, add some more control points to the map, and then re-georectify it to renew it, _and then_ use for your map.)

If you click on the 'Export' tab, you'll find that there's a URL for the map as tiles: `http://mapwarper.net/maps/tile/29435/{z}/{x}/{y}.png`.

That's the line we want to add to our map.

1. Open your `index.html` file.
2. We're going to add another layer, just after where you first specified your basemap, which to remind you, was at:

```
L.tileLayer('https://stamen-tiles-{s}.a.ssl.fastly.net/toner/{z}/{x}/{y}.png',
	{maxZoom: 19
}).addTo(myMap);
```

Hit 'enter' a few times after that final semicolon, and then paste in this:

```
L.tileLayer('http://mapwarper.net/maps/tile/29435/{z}/{x}/{y}.png',
  {maxZoom: 19}
).addTo(myMap);
```

Save the file, and then in your browser reload the page (I'm assuming you still have the server running as per step 8 in the previous section). Boom!

![result](/images/webmap/result.png)

But that map is a bit wonky, compared to our base map. Let's find a new map from Library and Archives Canada, georectify it, and then swap it in. (When you build your own map of your graveyard, you may or may not be able to find an underlying historical map. That's ok; if you can, great; if not, no worries.)

### Georectifying a base map

But as historians, we are often interested to show some process taking place in the past, so historical maps can be rhetorically extremely useful as base maps. But, to get such a map into a digital format that we can use, we need to georectify it.

Georectifying is the process of taking an image (whether it is of a historical map, chart, airphoto, or whatever) and manipulating its geometry so that it matches a geographic projection. Think of it like this: you take your handdrawn map, and use pushpins to pin down known locations on your map to a globe. As you pin, your image stretches and warps. Traditionally, this has not been an easy thing to do, if you are new to GIS. In recent years, the curve has flattened significantly. In this exercise, we'll grab an image, upload it to the Map Warper website, and then export it as a tileset which can be used in other mapping programs.

At [Library and Archives Canada](http://collectionscanada.gc.ca/pam_archives/index.php?fuseaction=genitem.displayEcopies&lang=eng&rec_nbr=98246&title=Laura%20A.%20Gamble%20fonds%20%5btextual%20record,%20graphic%20material%5d.&ecopy=e002743396) I found an "Insurance Plan of Ottawa, June 1878" [MIKAN 3824226](http://collectionscanada.gc.ca/lac-bac/results/images?module=images&SortSpec=score+desc&Language=eng&ShowForm=hide&SearchIn_1=mikanNumber&SearchInText_1=3824226&Operator_1=AND&SearchIn_2=&SearchInText_2=&Operator_2=AND&SearchIn_3=&SearchInText_3=&Level=&MaterialDateOperator=after&MaterialDate=&DigitalImages=1&Source=&ResultCount=10). (*nb* last time I checked, both of those links were borked at LAC's end; July 23 2021)

This one in particular [shows Parliament Hill](http://central.bac-lac.gc.ca/.item?app=fondsandcol&op=img&id=e010695477-v8). I will save this one to my machine and georectify it. You grab one that interests you!

1. Go to [Map Warper](http://mapwarper.net/) and sign up for an account. Then login.

2. Go to the upload screen:
![](https://i.imgur.com/bmNCzg6.png)

3. Fill in as much of the metadata as you can - it's important to credit the material correctly! Then select your map from your computer, and upload it.

4. On the next page, click 'Rectify'.
![](https://i.imgur.com/yULDRQR.jpg)

5. Pan and zoom both maps until you're sure you're looking at the same area in both. Double click in a map, select the location icon, and click on a point (location) you are sure you can match in the other window.

6. Click on the other map window, select the location icon, and then click on the same point.

7. Note the 'Add control point' button below and between both maps will light up. Click on this to confirm that this is a control point you want. Do this at least three times; the more times you can do it, the better the map warp.

8. Having selected your control points, click on 'Warp image'.

9. You can now click on the 'Export' panel, and get the URL for your georectified image in a few different formats. Copy the tiles based URL  and put that URL into your `index.html` file as you did in the previous section.

### Adding a layer control button

You can see that with leaflet it is pretty easy to add more layers and more data to your map. But soon things get a little bit unwieldy. We're going to re-organize our map so that we can turn layers on and off.

The end result will look like this:

![layer-control-button.png](/images/webmap/layer-control-button.png)

We're going to create some variables to hold our different layers and maps. Make a copy of your `index.html` file and call it `index2.html`. Open `index2.html` in Sublime Text and work on that file.

1. Delete the code from the `var myMap =` line to the `// load a GeoJSON from external file`. Your code should look like this now:

```
<body>
<div id="mapid" style="width: 600px; height: 400px;"></div>
<script>

  // load a GeoJSON from external file
```

We are going to create four variables that contain the different layers we want to show. Two are going to be base maps from Stamen (the 'toner' map and the 'watercolor' map), and two are going to be georectified maps we found on MapWarper showing downtown Ottawa. Insert after the `<script>` tag:

```
var toner = L.tileLayer('https://stamen-tiles-{s}.a.ssl.fastly.net/toner/{z}/{x}/{y}.png'),
  watercolor = L.tileLayer('https://stamen-tiles-{s}.a.ssl.fastly.net/watercolor/{z}/{x}/{y}.png'),
  historical1897 = L.tileLayer('https://mapwarper.net/maps/tile/29435/{z}/{x}/{y}.png'),
  lumberdistrict = L.tileLayer('https://mapwarper.net/maps/tile/27671/{z}/{x}/{y}.png')
```

The `L.tileLayer` tells leaflet the sources for our layers.

Now we're going to create two more variables to _group_ our layers into base maps and overlay maps:

```
var baseMaps = {
  "Toner": toner,
  "Watercolor": watercolor
};

var overlayMaps = {
  "1897": historical1897,
  "Lumber District": lumberdistrict
}
```

Next, we create a variable called `myMap` to tell Leaflet the centre of our map, the zoom layer, and the default base map; we pass this information to the html container 'mapid' so that our creation can be displayed:

```
var myMap = L.map('mapid', {
  center: [45.4192857, -75.6973237],
  zoom: 13,
  layers:[toner]
});
```

and finally, we add a control box so that we can switch the different layers on and off, and that our layers are grouped appropriately:

```
L.control.layers(baseMaps, overlayMaps).addTo(myMap);
```

Save that as `index2.html`, and with your server still running, you can view it in your browser at `http://localhost:8000/index2.html`.  (NB: the part after the colon might be different on your machine - this number is the 'port' and the precise port number will have printed out in your terminal when you first ran the `python -m http.server` command.)

For more on layers with leaflet, see [Layer Groups and Layers Control](https://leafletjs.com/examples/layers-control/).

### The Graveyard

Now that you've seen how to build a map and serve it online, let's modify it to show the graveyard you worked on.

+ You'll want the map to be centred on the graveyard you worked on, so you need to figure out the coordinates for it. These will go in the code for the variable `myMap`, and are in decimal degrees.
+ Coordinates for your stones go in the geojson file. Easiest way to do this might be to make a csv table like so, in a text editor like [Sublime Text](https://www.sublimetext.com/):
```
latitude,longitude,geometry,Label
45.4192857,-75.6973237,Point,Centre of our map!
45.4275,-75.699222,Point,Not the centre of the map
```
and then use [this tool](http://www.convertcsv.com/csv-to-geojson.htm) to convert it to geojson.
+ You can try to find a historical map for an underlay, but only if you're feeling keen.
