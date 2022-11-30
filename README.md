Get it on gitlab

https://gitlab.com/duplisea/ecan.map


# What it does

This is essentially a wrapper package to PBSmapping with functions that
reflect common features that a DFO scientist would want on a map such
as: survey strata, bathymetry data, trawl tracks and various colour
options. The maps are functionally useful if not the most beautiful (you
will make much more beautiful maps with sf, raster, ggmap or any number
of other packages), but they are expressive of the essential information
most scientists want in a map. They are usually a lot simpler to make
than most of those other packages. It is also quite fast to make the
maps and they save well as vector graphics (svg) with small file size
and zoomable without pixelation.

Perhaps even of greater utility than the simplicity is that datafiles
for various statistical areas for eastern Canada are made available in
the package and can be overlayed on the base map. This includes NAFO
areas, survey strata, some fishing zones. Please send me any you have as
a shapefile and I will include it here. Bathymetry data for eastern
Canada is also included.

It can be useful compared to arcgis, for example, because you can
integrate the map easily with your analysis code in R rather than
exporting files and making maps in another program. In this way, you can
easily create maps to visualise your data even in your initial data
explorations. You might then go to another R package, arcgis or grass to
make prettier maps if that is what you want but at least this might help
you at the start.

Maps are built up. First you call the basic map and then you add
features like bathymetry, statistical areas, trawl event data (that you
bring yourself). The order in which you do this will affect the final
map. For example, survey strata and bathymetry are redundant for certain
isobaths so if you plot the strata and then bathymetry, the bathmetric
lines will cover the strata lines. Likewise, you will probably want to
plot trawl event data as the last step because if you add bathmyetry
after the trawl data, the bathymetric lines will likely plot over the
top of some of your trawl survey positions.

Finally, you can get more fine grained control and use all the functions
in PBSmapping here. PBSmapping is the only dependency so hopefully this
will keep people out of dependency hell errors. I verbatim copied
functions from the package shape (of course with attribution) to prevent
a further dependency.

# What it does not do

This is aimed at producing maps that cover areas of many thousands of
square kilometres. If you zoom in on really small areas like bays, you
will find the coastline definition to be quite rough but it will still
work. If you want to do that and have smoother coastlines, you will need
very high resolution basemap data. If you have a shapefile with that
data, you can import it using “importShapefile” and then plotMap with
that imported shapefile. Depending on how that shapefile describes lat
and long, the other functions here may or may not work. You should make
your imported shapefile look like “worldLLhigh”, which is the datafile
creating the basemap here. You will then find that bathymetry data here
is too rough. I would suggest just using PBSmapping directly for such
cases as it is built more generally to accommodate your specific data
availability. Packages like raster allow you to import maps from google.
I am not really trying to hit that market, however, as my mapping needs
are modest.

Likewise, this package does not do a lot of GIS stuff but a lot of it is
there with the PBSmapping engine behind it. It does not import photos or
satellite images and there are no functions in the wrapper package
designed to go get data elsewhere.

# Installation

This package is stored on gitlab and is freely available.

    if (!require('devtools')) install.packages('devtools') # install devtools if you do not already have it
    devtools::install_gitlab("duplisea/ecan.map")
    library(ecan.map)

# Function conventions

The main functions of the package have suffix “.f”. There are other
functions as well but they are called by the .f functions. Most .f
functions can be called without arguments to produce a default map or
layer. Functions for tracks, arrows, and bubbles require the data that
you imported or a data specification so we cannot provide an
argumentless implementation: you need to specify your data. We have,
however, given examples of how to use them using mock data. Defaults are
usually set assuming the Gulf of St. Lawrence is the area of focus. You
can modify long and lat coordinates for your situation.

## Building up a map

The basic map with default options is focused on the Gulf of
St. Lawrence. This function will be the first step in any map’s
construction

    map.f()

![](README_files/figure-markdown_strict/basicmap-1.png)

You can change the colours of the land and sea to your choice. Super
ugly pumpkin example:

    map.f(land.colour="orange",sea.colour="black")

![](README_files/figure-markdown_strict/halloween-1.png)

### Show bathymetry

The next thing you might want is bathymetry. In this case we choose the
100m and 400m isobaths and we colour the isobath delimeters. We also put
the legend for bathmetry in the top left:

    map.f()
    bathymetry.f(isobaths=c(100,400), isob.col=c("slategrey","purple"), legend.pos="topleft")

![](README_files/figure-markdown_strict/bathymetry-1.png)

### Show particular defined areas

There is a data list in the package that has gathered up definitions of
various areas for example, survey strata and NAFO areas, that you may
want to show on your map. Presently, only survey strata for some regions
are available but the plan is to increase this to include all sorts of
areas such as NAFO, fishing areas, MPAs, EBSA …

So for this example let’s put survey strata (added progressively by DFO
region) on the map but this time we extend the default map to include
eastern Newfoundland shelf. We also do not plot bathymetry because it is
too busy and it is largely redundant with the survey strata which are
random stratified by depth.

    map.f(longs=c(-70,-40), lats= c(40, 60))
    stat.area.f("ngsl.trawl.survey.strata")
    stat.area.f("sgsl.trawl.survey.strata")
    stat.area.f("NL.trawl.survey.strata")
    stat.area.f("SS.trawl.survey.strata")

![](README_files/figure-markdown_strict/surveystrata-1.png)

You can also highlight particular strata by reference to the PID number
for it in the highlight.strata argument

    map.f()
    stat.area.f("gsl.EAR",strata.colour="lightblue",border.colour="black", highlight.strata=c(2,6), highlight.colour="red")

![](README_files/figure-markdown_strict/highlightstrata-1.png)

You can also show just your highlighted regions and no others by setting
“**show.only.highlight= T**”.

### Clipping and showing Ecosystem Approach Regions (EAR)

There is a polygon set definition of the [ecosystem approach regions
(EAR)](https://github/com/duplisea/gslea) developed for the Gulf of
St. Lawrence in this package. The EARs were initially defined for large
blocks that cover land as well but we want them only for the sea for
both plotting and area calculation.

    map.f()
    # we make the clip at -1500m which is in fact altitude because our file is a bathymetric definition 
    # and not topographic defintion and therefore altitude is -. Note that you can use this trick to show
    # mountains or notable topographic contours on land: use the function bathymetry.f with a negative isobath.
    EcoAppRegions.f(min.isobath=-1500, strata.colour="orange", border.colour="black")
    title("Original boundaries of EARs covering a lot of land")

![](README_files/figure-markdown_strict/bigEARs-1.png)

EARs are generally defined for offshore areas&gt;30 m depth (not all
data though) so we should clip the EAR definition by the 30m isobath
However, for some EARs like the Northumberland Strait, this eliminates
much of the EAR. To the contrary, if we clip it to just the shoreline
(0m isobath), the EAR definition then includes places that are not
relevant to the EAR like the lagoons in the Magdalen Islands. This is
not really a problem if one is interested in surface area of EARs
because these areas are small. It is more of a problem clipping to the
0m isobath from a visual presentation point of view because it almost
eliminates low lying areas like the Magdalen Islands from your map. You
might try then clipping to say 3m.

    map.f()
    EcoAppRegions.f(min.isobath=30, strata.colour="orange", border.colour="black")
    title("EARs clipped to 30m isobath")

![](README_files/figure-markdown_strict/cauliflowerEARs-1.png)

Contrast the map with the 30m clip to one with a 3m clip:

    map.f()
    EcoAppRegions.f(min.isobath=3, strata.colour="orange", border.colour="black")
    title("EARs clipped to 3m isobath")

![](README_files/figure-markdown_strict/niceEARs-1.png)

I have probably spent too much space on this because I have included the
EAR 3m clip as one of the statistical areas in the list containing them
all (type **names(statistical.areas)**). At least the function
*EcoAppREgions.f* will show you how to clip polygons to bathymetry for
your own boundaries.

### Show events (like survey sets)

You may want to plot all survey positions or fishing positions. The set
file for the Northern Gulf summer survey is included here to give you an
example of how to plot positions. You first need to convert the long and
lat positions to decimal-degrees.

    # decimal degrees function specific to how ngulf survey records position, this is not robust code.
    # packages like raster and sf will have more robust code to convert decimal degrees. You can also
    # convert online at various websites.
    decdeg= function(degmin.sec){
      # convert geographic coordinates in degree, minutes, decimal seconds into decimal degrees
      # in one number e.g. 44 degrees, 38 minutes, decimal seconds 73", input as 4438.73
      deg= floor(degmin.sec/10000)
      min= floor((degmin.sec - deg*10000)/100)/60
      decimal.remainder=10*degmin.sec%%1 # modulo 1, %%1, gives you the digits after the decimal
      dd= deg+min+decimal.remainder
      dd
    }
    x= -decdeg(ngulf.survey.positions$lon_deb*100)
    y= decdeg(ngulf.survey.positions$lat_deb*100)

    # The basemap and then use points to show all the survey positions for all years of the survey
    map.f()
    points(x,y,pch=".")

![](README_files/figure-markdown_strict/events-1.png)

It gives you a pretty good idea of areas that are not trawled like the
Mecatina Trough in the northeast and east of Anticosti Island (lots of
tear-ups in both places).

#### Add events like bubbles (e.g. catch at different locations)

This is functionality directly in PBSmapping so there is not a function
wrapper here to do it. We point you to the addBubbles function to do
this. It makes bubbles and legends that look nice. It can be a bit
iterative to figure out what works best with your data. Event data with
a bubble needs one more column (Z) to describe the magnitude of catch or
whatever it is you are trying to show. X and Y are the long and lat
positions. PBSmapping also requires that you make this an event class
object. It essentially means you have an event id (EID) associated with
each row. EID is just a sequential numbering from 1 to the end of your
file (number of rows).

Here we just randomly sample 20 set positions from the northern gulf
survey file, we convert them to decimal degree again (this makes the X
and Y columns). We then create the Z column by random draws from a
log-normal distribution. Your Z would actually be the biomass per set or
abundance per set or whatever it is that you want to show.

    # randomly sample the positions of 20 sets from the ngulf survey set file and convert to decimal degrees
    event.positions= sample(1:length(ngulf.survey.positions$lon_deb),20)
    x= ngulf.survey.positions$lon_deb[event.positions]
    y= ngulf.survey.positions$lat_deb[event.positions]
    x= -decdeg(x*100)
    y= decdeg(y*100)

    # makeup Z values by sampling from a log-nromal distribution and I wanted some zero values
    z= rlnorm(20,log(10),log(5))
    z[z<3]=0 # make values below 3 equal to 0

    # Make it event data. Note that EID is just a vector from 1 to the number of observations. 
    # Your analysis would likely start here after you imported your data.
    your.data= data.frame(EID=1:20, X=x, Y=y, Z=z) # this is what your data.file should look like
    fishing.events= as.EventData(your.data)

    # plot the base map
    map.f()

    # show the events using the addBubbles function from PBSmapping
    addBubbles(fishing.events,type="perceptual", symbol.bg=rgb(.9,.5,0,.6), legend.type="nested", symbol.zero="+", col="grey")

![](README_files/figure-markdown_strict/eventbubbles-1.png)

### Show trawl tracks or events with start and end positions

You can add survey tracks and arrows if you have start and end positions
of tows. Made up trawl tracks are used here

    #makeup fake tracks. They are really long tracks just for illustrative purposes
    tmp= data.frame(xstart= runif(10,-62,-60),ystart= runif(10,47.5,48.5))
    tmp$xend=tmp$xstart*1.01
    tmp$yend=tmp$ystart*1.01 

    # Call the base map
    map.f()

    # Add the tracks with a colour gradient
    tracks.f(tmp$xstart, tmp$ystart, tmp$xend, tmp$yend,start.colour="red", end.colour="blue", number.of.colours=30, lwd=1.5)

    # add arrowheads to the tracks
    track.arrows.f(tmp$xstart, tmp$ystart, tmp$xend, tmp$yend, arr.lwd=.06, arr.length=0.15, arr.col="blue", col="blue", arr.type="triangle")

![](README_files/figure-markdown_strict/tracks-1.png)

### All together (unmake this)

Finally a really busy map that you would never use but you can unmake it
to the degree you want by dropping off layers. You will likely have a
script that stepwise adds things like you see below. Strata are usually
the first thing to add because they are polygons that cover everything
that came before them. Some place names have been added using text to
show you how you can add them.

    map.f()
    stat.area.f("ngsl.trawl.survey.strata")
    stat.area.f("sgsl.trawl.survey.strata")
    bathymetry.f(isobaths=c(25,400),legend.pos="topleft")
    points(x, y, pch=".")
    tracks.f(tmp$xstart, tmp$ystart, tmp$xend, tmp$yend,start.colour="red", end.colour="blue", number.of.colours=30)
    track.arrows.f(tmp$xstart, tmp$ystart, tmp$xend, tmp$yend, arr.lwd=.06, arr.length=0.15, arr.col="blue", 
                   col="blue", arr.type="triangle")
    text(-55.5,48.5, labels="Province of Newfoundland and Labrador", cex=0.5)
    text(-70,48.5, labels="Québec", srt=45)

![](README_files/figure-markdown_strict/unmakethis-1.png)

# Additions

Please contribute data for your statistical areas or management areas.
Other sorts of functionality or wrappers are also welcome with the
provision that they do not create dependencies (i.e. do not call the
tidyverse) and which work with simple layering ideas.

# List of functions

Help files are available for these functions

-   map.f
-   stat.area.f
-   EcoAppRegions.f
-   bathymetry.f
-   tracks.f
-   track.arrows.f
-   hv.distance.f

# Note

If you are saving graphics files as vector graphics (svg), vector
graphics do not do well with colour gradients (files get big) because
they have to define a polygon for each colour block in the image. If you
have trawl tracks with a colour gradient and many tracks, you should
reduce the number of gradients (possibly to 1, i.e. only 1 colour) and
this should reduce your file size. If you save them to a raster format
(e.g. png), the basic file size will be larger than the svg with one
colour but it will not get larger with many gradient colours since a
pixel is a pixel be it green or red. That is why photographs are rarely
written as vector graphics but they are great for most scientific
graphs. Submitting papers to journals with vector graphics means that
they do not keep coming back to you looking for raster graphics at high
resolution and it makes your submitted manuscript package small.
