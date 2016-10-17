---
layout: post
title:  "Tutorial part 1: Create beautiful hillshade maps from digital elevation models with GDAL and Mapnik"
date:   2011-02-05 17:15:11 +0000
categories: tutorials
---


Mapnik is your best friend if you want to create map tiles for a slippy map or just want an open-source way to output high-quality printed maps for a wide variety of uses.

Though for your first project I might recommend a somewhat lower-priority project, <a href="http://media.mikejcorey.com/blog/2011/02/SaveTheDate3-5x5.jpg">I like to think I'm the only person to ever use Mapnik to make wedding invitations</a>.

(If I'm wrong, I'd love to see more examples!)

One of Mapnik's greatest strengths is its ability to combine multiple types of data into a single map: shapefiles, PostGIS data and raster data are all easy to combine.

When I began working with GIS data I started with exclusively vector data, because vector data are so easily scaled to any size and don't take up much storage space. Vector layers are also easy to color and style, and are great at keeping your thematic maps clean and easy to read.

But sooner or later you'll want to add some texture to your maps, and it's time to add elevation models with raster data.

Luckily, there's lots of raster data available to get started. The <a href="http://edc2.usgs.gov/geodata/index.php">USGS publishes digital elevation models (.DEM) of most of the United States</a> (everywhere except Alaska, I believe). This is definitely high-enough resolution for adding texture to a state-level visualization.

Less luckily, the digital elevation models are sliced up into small pieces. This can be nice because it cuts down on file sizes, but if you want a statewide elevation map, you'll have to put many pieces together.

A statewide elevation model of California, for example, requires <a href="http://www.brenorbrophy.com/California-DEM.htm">merging 70 .dem files at the 2 arc-second resolution</a>.

<a href="http://www.brenorbrophy.com/California-DEM.htm"><img class="aligncenter size-full wp-image-374" title="ca-components" src="http://media.mikejcorey.com/blog/2011/02/ca-components.jpg" alt="" width="610" height="610" /></a>

However, never fear! GDAL is here to help.

<a href="http://www.gdal.org/">GDAL is a powerful, open-source library of translation scripts</a> for raster geographic data.

So with GDAL, a shapefile of California's border and a set of digital elevation models and Mapnik, it's straightforward to make a seamless high-resolution .PNG map of California with nice elevation hillshades.

This tutorial isn't for people without any GIS or programming experience, but if you have a little of both you'll be fine. This tutorial is for Mac OS X Snow Leopard users, but will probably work for Linux users with a few OS-related tweaks.

Using GDAL requires knowledge of basic GIS concepts and basic command-line programming. I've had good luck installing the <a href="http://www.kyngchaos.com/software/frameworks">GDAL Complete package from kyngchaos.com</a>. Scroll down to the "GDAL 1.8 Complete" .dmg installer.

To get started with raster data, you mostly just need to know what a TIF image is.

To do any Mapnik work, you should feel comfortable with:
<ul>
	<li> GIS concepts like map projections</li>
	<li> Shapefiles and/or PostGIS</li>
	<li> XML</li>
	<li> Basic command-line programming</li>
</ul>
The Mapnik website has detailed <a href="http://trac.mapnik.org/wiki/MacInstallation">installation guides</a> and  <a href="http://trac.mapnik.org/wiki/MapnikTutorials">tutorials on how to use it</a>. There's an excellent <a href="http://mojodna.net/2009/12/05/the-os-x-spatial-stack.html">OS X-specific guide to installing Mapnik and dependencies (including GDAL and many other useful spacial libraries) here</a>. There's many ways to do it and I won't get into that here. Just make sure you have Mapnik running with GDAL support.

The tutorial is divided into two parts. Part 1, this post, will focus on merging, converting, projecting and clipping the digital elevation models so they can be used with Mapnik. We'll handle the Mapnik process in part 2.
<h2>Data mis en place</h2>
Here's the data we'll use for this tutorial:<a href="http://media.mikejcorey.com/download/ca-mercator.tar.gz"></a>
<ul>
	<li><a href="http://media.mikejcorey.com/download/ca-mercator.tar.gz">A California border shapefile in Mercator projection (SRID 3395)</a></li>
	<li> <a href="http://media.mikejcorey.com/download/raw-ca-dems.tar.gz">A g-zipped archive of all the digital elevation models you'll need for California</a></li>
</ul>
<h2>Fun with GDAL</h2>
Download and upzip the shapefile and the .dem folder. Then open a terminal window and navigate to wherever you downloaded the shapefile and dem folders.

Then, in the terminal:

{% highlight bash %}
$ cd raw-ca-dems
{% endhighlight %}

First we'll merge the digital elevation model tiles into a single .dem file using <strong>gdal_merge</strong>. We need to specify that we want any areas without data to be white by specifying <strong>-init "255"</strong>. Then we specify our desired output filename with <strong>-o (filename)</strong> and then list the files we want to merge together. Like so:

{% highlight bash %}
$ gdal_merge.py -init "255" -o ca-dem-combined.dem alturas-e.dem alturas-w.dem bakersfield-e.dem bakersfield-w.dem chico-e.dem chico-w.dem crescent_city-e.dem death_valley-e.dem death_valley-w.dem el_centro-e.dem el_centro-w.dem eureka-e.dem fresno-e.dem fresno-w.dem goldfield-w.dem kingman-e.dem kingman-w.dem klamath_falls-w.dem las_vegas-w.dem long_beach-e.dem long_beach-w.dem los_angeles-e.dem los_angeles-w.dem lovelock-w.dem mariposa-e.dem mariposa-w.dem medford-e.dem medford-w.dem monterey-e.dem monterey-w.dem needles-e.dem needles-w.dem noyo_canyon-e.dem redding-e.dem redding-w.dem reno-e.dem reno-w.dem sacramento-e.dem sacramento-w.dem salton_sea-e.dem salton_sea-w.dem san_bernardino-e.dem san_bernardino-w.dem san_clemente_island-e.dem san_diego-e.dem san_diego-w.dem san_francisco-e.dem san_francisco-w.dem san_jose-e.dem san_jose-w.dem san_luis_obispo-e.dem san_luis_obispo-w.dem santa_ana-e.dem santa_ana-w.dem santa_cruz-e.dem santa_maria-e.dem santa_rosa_island-e.dem santa_rosa-e.dem santa_rosa-w.dem susanville-e.dem susanville-w.dem trona-e.dem trona-w.dem ukiah-e.dem ukiah-w.dem vya-w.dem walker_lake-e.dem walker_lake-w.dem weed-e.dem weed-w.dem
{% endhighlight %}

As long as you don't get any errors, you should now have a new file in your folder called ca-dem-combined.dem. If you have GIS software like Quantum GIS you should be able to add this as a raster layer. If you do open it up, you shouldn't see any seams between the former tiles -- that's what we want for a good-looking Mapnik layer.

But for this to work in Mapnik, we need a GeoTIFF. To convert the merged .dem to a GeoTIFF, we'll use <a href="http://www.gdal.org/gdaldem.html"><strong>gdaldem</strong></a>. The gdaldem command has several modes; we'll use hillshade to create a shaded relief map, the most common way to visualize texture. We also need to specify the ratio of vertical units to horizontal. Since right now the .dem is in WGS84 projection (standard latitude/longitude), we'll use degrees, so <strong>-s 111120</strong>. Then we specify the input file and the output file. Here's the full command:

{% highlight bash %}
$ gdaldem hillshade -s 111120 ca-dem-combined.dem ca-dem-combined.tif
{% endhighlight %}

Even if you don't have a GIS viewer, you should be able to open the .tif file in any image browser like Preview or Photoshop. (Be careful you don't resave the file, though -- most image editors will remove necessary geometric data from the file if you re-save it.) You'll see a few thin black lines around the outside edge of the old tile data, but don't worry -- we'll cut this out later.

<img class="aligncenter size-full wp-image-378" title="ca-combined" src="http://media.mikejcorey.com/blog/2011/02/ca-combined.jpg" alt="" width="620" height="620" />

So now we have a combined .tif, but most of the time we want to display our finished project in Mercator projection -- most non-GIS types will think non-Mercator maps look a bit strange. We need to reproject our .tif to Mercator with <a href="http://www.gdal.org/gdalwarp.html"><strong>gdalwarp</strong></a>. We'll use the <strong>-t_srs</strong> option to set the target projection, and use the projection specification for the version of Mercator that's also used by Google Maps and OpenLayers.

{% highlight bash %}
$ gdalwarp -t_srs '+proj=merc +lon_0=0 +k=1 +x_0=0 +y_0=0 +ellps=WGS84 +datum=WGS84 +units=m +no_defs' ca-dem-combined.tif ca-dem-combined-merc.tif
{% endhighlight %}

If you open up the resulting file, you should see that the projection has changed.

<img class="aligncenter size-full wp-image-379" title="ca-merc" src="http://media.mikejcorey.com/blog/2011/02/ca-merc.jpg" alt="" width="620" height="781" />

Now we have our final hillshade data, but we want to only show California -- not all the extra stuff outside the border. We'll use our California border shapefile as a mask to cut out just the hillshade data inside the boundary. This is a three-step process.

First, we determine the bounding box, or extent, of our clipping mask -- the California border shapefile in this case. We use <strong>ogrinfo</strong> for this, which comes along with GDAL Complete.

{% highlight bash %}
$ ogrinfo -al ../ca-mercator/ca-mercator.shp
{% endhighlight %}

This will return a lot of details about the shapefile. But if you scroll back to the top of the output you'll see:

{% highlight bash %}
Extent: (-13849389.898804, 3810165.061203) - (-12704836.275367, 5133847.169980)
{% endhighlight %}

These represent the longitude and latitude points (in Mercator projection) of the southwest and northeast corners of our shapefile data.

Next, we trim our merged Mercator .tif to those exact boundaries with <a href="http://www.gdal.org/gdal_translate.html"><strong>gdal_translate</strong></a>.

{% highlight bash %}
$ gdal_translate -projwin -13849389.898804 5133847.169980 -12704836.275367 3810165.061203 ca-dem-combined-merc.tif ca-dem-combined-merc-box.tif
{% endhighlight %}

<strong>Warning:</strong> Notice that in the above command we reversed the order of the two y-axis values (5133847.169980 and 3810165.061203) from how they appeared in our ogrinfo query. This is because gdal_translate's <strong>-projwin</strong> option is looking for the upper left and lower right coordinates -- exactly the opposite of the extent we got from ogrinfo. Annoying, but that's just the way it is.

Finally, we'll use the shapefile's polygon boundaries as a clipping mask on the merged, projected and trimmed .tif, which is now trimmed to show the same area as the shapefile. We'll use <a><strong>gdalwarp</strong></a> for this, using some different options than before. The <strong>-co COMPRESS=DEFLATE</strong> option is a generic "creation option" that can have many values. We specify <strong>-dstalpha</strong> to create a "nodata" band in our resulting .tif, so we are left with a transparent background. And finally we use <strong>-cutline</strong> to specify a file to use as the clipping mask. As before we then specify our input file and desired output filename.

{% highlight bash %}
$ gdalwarp -co COMPRESS=DEFLATE -dstalpha -cutline ../ca-mercator/ca-mercator.shp ca-dem-combined-merc-box.tif ca-dem-combined-merc-cutout.tif
{% endhighlight %}

And there's our textured California border. If you open <strong>ca-dem-combined-merc-cutout.tif</strong> in an image viewer, you should see a beautifully clipped grayscale image.

<img class="aligncenter size-full wp-image-380" title="ca-cutout" src="http://media.mikejcorey.com/blog/2011/02/ca-cutout.jpg" alt="" width="620" height="717" />

<a href="/tutorials/2011/02/25/Tutorial-part-2-Create-beautiful-hillshade-maps-from-digital-elevation-models-with-GDAL-and-Mapnik.html">In the second half of our tutorial, we'll use this cutout .tif as a layer in Mapnik</a>. We're more than halfway there!
