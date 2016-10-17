---
layout: post
title:  "Tutorial part 2: Create beautiful hillshade maps from digital elevation models with GDAL and Mapnik"
date:   2011-02-25 17:15:11 +0000
categories: tutorials
---

<a href="/tutorials/2011/02/05/Tutorial-part-1-Create-beautiful-hillshade-maps-from-digital-elevation-models-with-GDAL-and-Mapnik.html">In part 1 of our GDAL and Mapnik hillshade map tutorial</a>, we used GDAL to convert tiled USGS digital elevation models to a merged GeoTIFF. When also reprojected the map to Mercator and used a California border shapefile to cut out just the state of California.

In this installment, we'll use the result as a Mapnik layer, and check out some different options for using raster data in your Mapnik maps.

Before we dive in, a note about versions. In this tutorial I'm using Mapnik2, which is now in beta release. I'm doing this both because Mapnik2 adds some exciting new capabilities, and so this tutorial is worth anything for more than a few weeks. Even though Mapnik2 is still beta, I've been running it for several weeks now and so far haven't run into any serious difficulties, so especially if you're on Mac OS X, I'd recommend giving version 2 a shot.

Even if you do run into problems, <a href="http://www.mail-archive.com/mapnik-users@lists.berlios.de/info.html">the Mapnik listserv is very active</a>, and the developers themselves are very quick to respond with help and to fix bugs. If only every open-source project was so well-maintained!

If you're not running version 2, a lot of this will still work, though you might need to tweak syntax a little bit. I'll note anything you just can't do in Mapnik before version 2.

<strong>Data mis en place</strong>

Here's the data we'll use for this part of the tutorial:

<ul>
	<li><a href="media.mikejcorey.com/download/ca-mercator.tar.gz">A California border shapefile in Mercator projection (SRID 3395)</a></li>
	<li><a href="media.mikejcorey.com/download/california_county_shortline.tar.gz">A California county/shorelines shapefile (SRID 4326)</a></li>
	<li><a href="media.mikejcorey.com/download/california_water.tar.gz">A California water shapefile (SRID 4269)</a></li>
	<li><a href="media.mikejcorey.com/download/ca-dem-combined-merc-cutout.tif.gz">The final GeoTIFF cutout from part 1</a> (This is an 88 MB file, so don't download if you don't have to!)</li>
</ul>


Now on to the fun!

First, let's do a basic Mapnik map with our GeoTIFF layer. For many Mapnik maps, <a href="https://github.com/mapnik/Cascadenik/wiki/Cascadenik">Cascadenik makes styling much simpler for anyone familiar with CSS syntax</a>, but currently (February 2011) Cascadenik doesn't support raster layers. So we'll start with this XML:

{% highlight xml %}
<Map srs="+proj=merc +lon_0=0 +k=1 +x_0=0 +y_0=0 +ellps=WGS84 +datum=WGS84 +units=m +no_defs">
  <Style name="hillshade style">
    <Rule name="rule 1">
      <RasterSymbolizer opacity="1" scaling="bilinear" mode="normal" />
    </Rule>
  </Style>
  <Layer name="hillshade">
    <StyleName>hillshade style</StyleName>
    <Datasource>
      <Parameter name="type">gdal</Parameter>
      <Parameter name="file">/Users/yourname/Documents/gdaltutorial/ca-dem-combined-merc-cutout.tif</Parameter>
    </Datasource>
  </Layer>
</Map>
{% endhighlight %}


Let's break this down in stages. First, the overall <strong>&lt;Map&gt;</strong> wrapper:

{% highlight xml %}
<Map srs="+proj=merc +lon_0=0 +k=1 +x_0=0 +y_0=0 +ellps=WGS84 +datum=WGS84 +units=m +no_defs">
</Map>
{% endhighlight %}

This is where we set the projection of the overall map. Don't worry, you don't need to understand what all of it means (although it's fun to find out!). As you no doubt noticed, this is the same srs data we used to reproject our GeoTIFF in part 1.

One of the great features of Mapnik is that even if the data for separate layers are in different projections, in most cases Mapnik can simply reproject that data on the fly. You can't reproject raster data within Mapnik, however, so you'll always want to be sure that you get your GeoTIFF into the projection you want your final map to use before you get to Mapnik (as we did in part 1).

Now let's drop down a bit to the layer data:

{% highlight xml %}
  <Layer name="hillshade">
    <StyleName>hillshade style</StyleName>
    <Datasource>
      <Parameter name="type">gdal</Parameter>
      <Parameter name="file">/Users/yourname/Documents/gdaltutorial/ca-dem-combined-merc-cutout.tif</Parameter>
    </Datasource>
  </Layer>
{% endhighlight %}

This is where we tell Mapnik what type of data we're using for this layer. Inside the <strong>&lt;Datasource&gt;&lt;/Datasource&gt;</strong> tags, we specify the data type (<strong>gdal</strong>), and point to the location of our GeoTIFF file. It's good practice to use the absolute path to your files so if you move your Mapnik files you won't break the script.

The remaining line inside our <strong>&lt;Layer&gt;&lt;/Layer&gt;</strong> tags is the <strong>&lt;StyleName&gt;&lt;/StyleName&gt;</strong>. This tells Mapnik which style definition to use for this layer. So let's take a look at that style definition.

{% highlight xml %}
  <Style name="hillshade style">
    <Rule name="rule 1">
      <RasterSymbolizer opacity="1" scaling="bilinear" mode="normal" />
    </Rule>
  </Style>
{% endhighlight %}

There's a lot you can do with raster data, especially in Mapnik2. This style definiton is pretty basic, though. We're saying that we want the GeoTIFF to be rendered at 100% opacity. Scaling controls how Mapnik renders the image when scaling it down from its original size, and the mode controls how the raster layer is blended with layers below it. Since there's nothing currently behind the layer, mode is somewhat moot at the moment. <a href="http://trac.mapnik.org/wiki/RasterSymbolizer#Usage"</a>Feel free to play with different values for both scaling and mode.</a>

That's about it. Just make sure your style name matches the <strong>&lt;StyleName&gt;&lt;/StyleName&gt;</strong> tag in your layer.

That's it for our first, pretty basic GeoTIFF Mapnik map. Save this file as <strong>ca-map.xml</strong>.

Now we'll use Python to compile this XML into an image. I highly recommend that once you get the basics down, you simplify your life and download <a href="https://github.com/mapnik/Cascadenik/wiki/Cascadenik">Cascadenik</a> and <a href="http://code.google.com/p/mapnik-utils/wiki/Nik2Img">nik2img</a> to do a little of this coding for you, but at this point you've installed enough for one tutorial. So we'll stick to a basic Python script. Even if you've never used Python before, now's the time to start. Trust me, if you have any programming experience this will make sense.

Open a new text file, and save the following code as <strong>ca-stuff-compile.py</strong> into the same directory as your XML document:

{% highlight bash %}
#!/usr/bin/env python
import mapnik2
mapfile = 'ca-stuff.xml'
map_output = 'ca-map.png'
m = mapnik2.Map(1200, 800)
mapnik2.load_map(m, mapfile)
bbox = mapnik2.Box2d(mapnik2.Coord(-14000000, 3700000), mapnik2.Coord(-12500000, 5200000))
m.zoom_to_box(bbox)
mapnik2.render_to_file(m, map_output)
{% endhighlight %}

Right off the bat, if you're not using Mapnik2 for this, you'll need to change everthing that says <strong>mapnik2</strong> to simply <strong>mapnik</strong>.

Next we assign some variables. We specify our XML file for Mapnik to chew on, and tell Mapnik what filename to use for the final image.

{% highlight bash %}
mapfile = 'ca-stuff.xml'
map_output = 'ca-map.png'
{% endhighlight %}


Next we instantiate a new mapnik map, and pass our XML to Mapnik's load_map() function. We also specify the pixel dimensions we want to give our final map, 800 pixels wide by 600 pixels deep.

{% highlight bash %}
m = mapnik2.Map(800, 600)
mapnik2.load_map(m, mapfile)
{% endhighlight %}

Now we tell Mapnik what bounding box we want the map to use. We'll use this to control how much space will show up around our map content, but you could also specify only a section of the map or any other arbitrary bounding box. But how do we know what to use?

Remember back in part 1 when we used <strong>ogrinfo</strong> to get the bounding box of our California shapefile that we used to cut out the GeoTIFF? Here's what that returned:

{% highlight bash %}
Extent: (-13849389.898804, 3810165.061203) - (-12704836.275367, 5133847.169980)
{% endhighlight %}

So if you want to clip your map at California's exact boundaries, you can use that extent exactly. But generally we want our maps to have a little padding, so we'll round up and down a bit:

{% highlight bash %}
bbox = mapnik2.Box2d(mapnik2.Coord(-14000000, 3700000), mapnik2.Coord(-12500000, 5200000))
m.zoom_to_box(bbox)
{% endhighlight %}

Note that, mercifully, Mapnik uses the same format for its bounding box as <strong>ogrinfo</strong>. The points represent the lower left and upper right corners of our image.

Finally, the script renders the map to our file:

{% highlight bash %}
mapnik2.render_to_file(m, map_output)
{% endhighlight %}

If you haven't already save this file into the same directory as your XML document as <strong>ca-stuff-compile.py</strong>.

Now, in a terminal window, change into this directory, and let's build that map:

{% highlight bash %}
$ python ca-stuff-compile.py
{% endhighlight %}

If you don't get any error messages, check your directory. You should see a new file called ca-map.png, and hopefully your hillshade map is in the middle.

<img src="http://media.mikejcorey.com/blog/2011/02/ca-mapnik-1.jpg" alt="" title="ca-mapnik-1" width="610" height="406" class="aligncenter size-full wp-image-400" />

Neato! Well, sort of - it's pretty boring, right? Not much different from our original GeoTIFF. Let's change that now by adding more styles and more map layers to our XML. We'll add a lot at once here, but not so much conceptually:

{% highlight xml %}
<Map background-color="rgb(51,122,207)" srs="+proj=merc +lon_0=0 +k=1 +x_0=0 +y_0=0 +ellps=WGS84 +datum=WGS84 +units=m +no_defs">
  <Style name="hillshade style">
    <Rule name="rule 1">
      <RasterSymbolizer opacity="0.8" scaling="bilinear" mode="multiply" />
    </Rule>
  </Style>
  <Style name="polygon style">
    <Rule name="rule 1">
      <PolygonSymbolizer fill="rgb(255,204,102)" fill-opacity="1" />
    </Rule>
  </Style>
  <Style name="county style">
    <Rule name="rule 1">
      <LineSymbolizer stroke="#CCCCCC" stroke-width="0.4" />
    </Rule>
  </Style>
  <Style name="water style">
    <Rule name="rule 1">
      <PolygonSymbolizer fill="rgb(51,122,207)" fill-opacity="1" />
    </Rule>
  </Style>
  <Layer name="stateborder">
    <StyleName>polygon style</StyleName>
    <Datasource>
      <Parameter name="file">/Users/youruser/Documents/gdaltutorial/ca-mercator/ca-mercator.shp</Parameter>
      <Parameter name="type">shape</Parameter>
    </Datasource>
  </Layer>
  <Layer name="hillshade">
    <StyleName>hillshade style</StyleName>
    <Datasource>
      <Parameter name="type">gdal</Parameter>
      <Parameter name="file">/Users/youruser/Documents/gdaltutorial/ca-dem-combined-merc-cutout.tif</Parameter>
    </Datasource>
  </Layer>
  <Layer name="water" srs="+proj=longlat +ellps=WGS84 +no_defs">
    <StyleName>water style</StyleName>
    <Datasource>
      <Parameter name="file">/Users/youruser/Documents/gdaltutorial/california_water/california_water.shp</Parameter>
      <Parameter name="type">shape</Parameter>
    </Datasource>
  </Layer>
  <Layer name="counties" srs="+proj=longlat +ellps=GRS80 +datum=NAD83 +no_defs">
    <StyleName>county style</StyleName>
    <Datasource>
      <Parameter name="file">/Users/youruser/Documents/gdaltutorial/california_county_shoreline/california_county_shoreline.shp</Parameter>
      <Parameter name="type">shape</Parameter>
    </Datasource>
  </Layer>
</Map>
{% endhighlight %}

We've added 3 shapefile layers: one in Mercator projection that you may recognize as the state border shapefile we used to cutout the GeoTIFF in part 1:

{% highlight xml %}
  <Layer name="stateborder">
    <StyleName>polygon style</StyleName>
    <Datasource>
      <Parameter name="file">/Users/youruser/Documents/gdaltutorial/ca-mercator/ca-mercator.shp</Parameter>
      <Parameter name="type">shape</Parameter>
    </Datasource>
  </Layer>
{% endhighlight %}

You'll also notice that we've put this layer before the GeoTIFF in our XML. This is because we want the state border to be behind the GeoTIFF in the rendered image.

We've specified that this layer uses <strong>&lt;StyleName&gt;polygon style&lt;/StyleName&gt;</strong>, so let's take a look at the <strong>polygon style</strong> style, defined above the layers:

{% highlight xml %}
  <Style name="polygon style">
    <Rule name="rule 1">
      <PolygonSymbolizer fill="rgb(255,204,102)" fill-opacity="1" />
    </Rule>
  </Style>
{% endhighlight %}

This one's pretty simple. We set a fill color for features in this layer using standard Web RGB syntax and Mapnik's <a href="http://trac.mapnik.org/wiki/PolygonSymbolizer"><strong>PolygonSymbolizer</strong></a>, and specify that we want 100% opacity for this layer.

Next let's turn to the style for the GeoTIFF, which we've now changed so it will blend with our state border background layer:

{% highlight xml %}
  <Style name="hillshade style">
    <Rule name="rule 1">
      <RasterSymbolizer opacity="0.8" scaling="bilinear" mode="multiply" />
    </Rule>
  </Style>
{% endhighlight %}

We've changed the opacity of the GeoTIFF layer to 80%, to keep the level of gray down, and we've changed the blend mode to <strong>multiply</strong>. This tells Mapnik how to blend the GeoTIFF with the underlying layers. There are <a href="http://trac.mapnik.org/wiki/RasterSymbolizer#Usage">several other modes you can experiment with</a>, but in most cases you'll probably want multiply.

Now we'll add a water layer so we can see some of California's major lakes, and then county borders. Here's another great thing about Mapnik: You can use many types of data as layers: shapefiles, PostGIS tables and many other ogr types.

Here's the top 2 layers:

{% highlight xml %}
  <Layer name="water" srs="+proj=longlat +ellps=WGS84 +no_defs">
    <StyleName>water style</StyleName>
    <Datasource>
      <Parameter name="file">/Users/youruser/Documents/gdaltutorial/california_water/california_water.shp</Parameter>
      <Parameter name="type">shape</Parameter>
    </Datasource>
  </Layer>
  <Layer name="counties" srs="+proj=longlat +ellps=GRS80 +datum=NAD83 +no_defs">
    <StyleName>county style</StyleName>
    <Datasource>
      <Parameter name="file">/Users/youruser/Documents/gdaltutorial/california_county_shoreline/california_county_shoreline.shp</Parameter>
      <Parameter name="type">shape</Parameter>
    </Datasource>
  </Layer>
{% endhighlight %}

The main thing to notice about these layers is that we've specified an additional parameter in the <strong>&lt;Layer&gt;&lt;/Layer&gt;</strong> tag: <strong>srs</strong>. We're doing this because each of these layers is in a different spatial reference system than our main map. The water layer is in SRID 4326, or standard latitude/longitude pairs. The county layer is in SRID 4269, otherwise known as NAD83. As long as you specify the correct srs, Mapnik will do any reprojection needed on the fly (though not on raster layers, remember), so there's no need to reproject everything into the same projection on your own.

All that's left is the style definitions for those two layers:

{% highlight xml %}
  <Style name="county style">
    <Rule name="rule 1">
      <LineSymbolizer stroke="#CCCCCC" stroke-width="0.4" />
    </Rule>
  </Style>
  <Style name="water style">
    <Rule name="rule 1">
      <PolygonSymbolizer fill="rgb(51,122,207)" fill-opacity="1" />
    </Rule>
  </Style>
{% endhighlight %}

Nothing too fancy here. The county style is using a <a href="http://trac.mapnik.org/wiki/LineSymbolizer"><strong>LineSymbolizer</strong></a> instead of the PolygonSymbolizer we saw earlier. Look - we can use hexadecimal color codes too! And we set the stroke width to 0.4 pixels. The water style colors the water blue -- not much more to say than that.

OK, time to render this map. In your Terminal:

{% highlight bash %}
$ python ca-stuff-compile.py
{% endhighlight %}

If you did everything right, you should have a simple but pretty nice multi-layer Mapnik map.

<a href="http://media.mikejcorey.com/blog/2011/02/ca-mapnik-2.jpg"><img src="http://media.mikejcorey.com/blog/2011/02/ca-mapnik-2.jpg" alt="" title="ca-mapnik-2" width="610" height="407" class="aligncenter size-full wp-image-402" /></a>

Since we used a fairly high-resolution GeoTIFF, you should be able to change the image dimensions in ca-stuff-compile.py to make the image quite large for print, HD video, or a poster for your bedroom. But even I don't do that with my maps!

Unless you want to <a href="http://www.axismaps.com/typographic.php">send me this one</a>.
