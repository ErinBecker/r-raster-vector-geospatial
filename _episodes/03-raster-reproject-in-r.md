---
title: "Reproject Raster Data in R"
teaching: 10
exercises: 0
questions:
- "What to do when rasters don't line up."
objectives:
- "Be able to reproject a raster in `R`."
keypoints:
- ""
authors: [Leah A. Wasser, Megan A. Jones, Zack Brym, Kristina Riemer, Jason Williams, Jeff Hollister,  Mike Smorul, Joseph Stachelek]
contributors: [Michael Heeremans]
packagesLibraries: [raster, rgdal, ggplot2]
dateCreated:  2015-10-23
lastModified: 2018-06-19
categories:  [self-paced-tutorial]
tags: [R, raster, spatial-data-gis]
tutorialSeries: [raster-data-series]
mainTag: raster-data-series
description: "This tutorial explores issues associated with working with rasters
in different Coordinate Reference Systems (CRS) / projections. When two rasters
are in different CRS, they will not plot nicely together on a map. We will learn
how to reproject a raster in R using the projectRaster function in the raster
package."
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink:
comments: true
---



Sometimes we encounter raster datasets that do not "line up" when plotted or
analyzed. Rasters that don't line up are most often in different Coordinate
Reference Systems (CRS).

This tutorial explains how to deal with rasters in different, known CRSs. It
will walk though reprojecting rasters in `R` using the `projectRaster()`
function in the `raster` package.

## Raster Projection in R

In the [Plot Raster Data in R]({{ site.baseurl }}/R/Plot-Rasters-In-R/)
tutorial, we learned how to layer a raster file on top of a hillshade for a nice
looking basemap. In this tutorial, all of our data were in the same CRS. What
happens when things don't line up?

We will use the `raster` and `rgdal` packages in this tutorial.


~~~
# load raster package
library(raster)
~~~
{: .r}



~~~
Loading required package: sp
~~~
{: .output}



~~~
library(rgdal)
~~~
{: .r}



~~~
rgdal: version: 1.3-2, (SVN revision 755)
 Geospatial Data Abstraction Library extensions to R successfully loaded
 Loaded GDAL runtime: GDAL 2.1.3, released 2017/20/01
 Path to GDAL shared files: /Library/Frameworks/R.framework/Versions/3.5/Resources/library/rgdal/gdal
 GDAL binary built with GEOS: FALSE 
 Loaded PROJ.4 runtime: Rel. 4.9.3, 15 August 2016, [PJ_VERSION: 493]
 Path to PROJ.4 shared files: /Library/Frameworks/R.framework/Versions/3.5/Resources/library/rgdal/proj
 Linking to sp version: 1.3-1 
~~~
{: .output}



~~~
library(ggplot2)
library(dplyr)
~~~
{: .r}



~~~

Attaching package: 'dplyr'
~~~
{: .output}



~~~
The following objects are masked from 'package:raster':

    intersect, select, union
~~~
{: .output}



~~~
The following objects are masked from 'package:stats':

    filter, lag
~~~
{: .output}



~~~
The following objects are masked from 'package:base':

    intersect, setdiff, setequal, union
~~~
{: .output}

Let's create a map of the Harvard Forest Digital Terrain Model
(`DTM_HARV`) draped or layered on top of the hillshade (`DTM_hill_HARV`).


~~~
# import DTM
DTM_HARV <- raster("data/NEON-DS-Airborne-Remote-Sensing/HARV/DTM/HARV_dtmCrop.tif")
# import DTM hillshade
DTM_hill_HARV <- raster("data/NEON-DS-Airborne-Remote-Sensing/HARV/DTM/HARV_DTMhill_WGS84.tif")

# convert to data.frames
DTM_HARV_df <- DTM_HARV  %>% 
  rasterToPoints(., spatial = TRUE) %>% 
  data.frame()

DTM_hill_HARV_df <- DTM_hill_HARV  %>% 
  rasterToPoints(., spatial = TRUE) %>% 
  data.frame()

ggplot() +
     geom_raster(data = DTM_hill_HARV_df, 
                 aes(x = x, y = y, 
                   alpha = HARV_DTMhill_WGS84)
                 ) +
     geom_raster(data = DTM_HARV_df , 
             aes(x = x, y = y, 
                  fill = HARV_dtmCrop,
                  alpha=0.8)
             ) + 
     scale_fill_gradientn(name = "Elevation", colors = terrain.colors(10)) +
     guides(fill = guide_colorbar()) +
     scale_alpha(range = c(0.25, 0.65), guide = "none") +
    theme_bw() +
    theme(panel.grid.major = element_blank(), 
          panel.grid.minor = element_blank()) +
     theme(axis.title.x = element_blank(),
           axis.title.y = element_blank()) +
     ggtitle("Digital Terrain Model - NEON Harvard Forest Field Site") +
     coord_equal()
~~~
{: .r}

<img src="../fig/rmd-03-import-DTM-hillshade-1.png" title="plot of chunk import-DTM-hillshade" alt="plot of chunk import-DTM-hillshade" style="display: block; margin: auto;" />

Our results are curious - neither the Digital Terrain Model (`DTM_HARV_df`) 
or the DTM Hillshade (`DTM_hill_HARV_df`)did not plot on
Let's try to
plot the DTM on it's own to make sure there are data there.


~~~
ggplot() +
     geom_raster(data = DTM_HARV_df , 
             aes(x = x, y = y, 
                  fill = HARV_dtmCrop,
                  alpha=0.8)
             ) + 
     scale_fill_gradientn(name = "Elevation", colors = terrain.colors(10)) +
     guides(fill = guide_colorbar()) +
     scale_alpha(range = c(0.25, 0.65), guide = "none") +
    theme_bw() +
    theme(panel.grid.major = element_blank(), 
          panel.grid.minor = element_blank()) +
     theme(axis.title.x = element_blank(),
           axis.title.y = element_blank()) +
     ggtitle("Digital Terrain Model - NEON Harvard Forest Field Site") +
     coord_equal()
~~~
{: .r}

<img src="../fig/rmd-03-plot-DTM-1.png" title="plot of chunk plot-DTM" alt="plot of chunk plot-DTM" style="display: block; margin: auto;" />

Our DTM seems to contain data and plots just fine. 

Next we plot the DTM Hillshade on it's own to see whether everything is OK.


~~~
ggplot() +
     geom_raster(data = DTM_hill_HARV_df, 
                 aes(x = x, y = y, 
                   alpha = HARV_DTMhill_WGS84)
                 ) +
     guides(fill = guide_colorbar()) +
     scale_alpha(range = c(0.25, 0.65), guide = "none") +
    theme_bw() +
    theme(panel.grid.major = element_blank(), 
          panel.grid.minor = element_blank()) +
     theme(axis.title.x = element_blank(),
           axis.title.y = element_blank()) +
     ggtitle("Digital Terrain Model Hillshade - NEON Harvard Forest Field Site") +
     coord_equal()
~~~
{: .r}

<img src="../fig/rmd-03-plot-DTM-hill-1.png" title="plot of chunk plot-DTM-hill" alt="plot of chunk plot-DTM-hill" style="display: block; margin: auto;" />

Now we see the problem, the projections of the two rasters are different.
When this is the case, `ggplot` won't render the image, or unforunately,
throw an error message.

Let's next check the
 Coordinate Reference System (CRS) of the DTM and compare it to our hillshade.


~~~
# view crs for DTM
crs(DTM_HARV)
~~~
{: .r}



~~~
CRS arguments:
 +proj=utm +zone=18 +datum=WGS84 +units=m +no_defs +ellps=WGS84
+towgs84=0,0,0 
~~~
{: .output}



~~~
# view crs for hillshade
crs(DTM_hill_HARV)
~~~
{: .r}



~~~
CRS arguments:
 +proj=longlat +datum=WGS84 +no_defs +ellps=WGS84 +towgs84=0,0,0 
~~~
{: .output}

Aha! `DTM_HARV` is in the UTM projection. `DTM_hill_HARV` is in
`Geographic WGS84` - which is represented by latitude and longitude values.
Because the two rasters are in different CRSs, they don't line up when plotted
in `R`. We need to *reproject* `DTM_hill_HARV` into the UTM CRS. Alternatively,
we could project `DTM_HARV` into WGS84.

## Reproject Rasters
We can use the `projectRaster` function to reproject a raster into a new CRS.
Keep in mind that reprojection only works when you first have a *defined* CRS
for the raster object that you want to reproject. It cannot be used if *no*
CRS is defined. Lucky for us, the `DTM_hill_HARV` has a defined CRS.

> ## Data Tip
> When we reproject a raster, we
> move it from one "grid" to another. Thus, we are modifying the data! Keep this
> in mind as we work with raster data.
{: .callout}

To use the `projectRaster` function, we need to define two things:

1. the object we want to reproject and
2. the CRS that we want to reproject it to.

The syntax is `projectRaster(RasterObject, crs = CRSToReprojectTo)`

We want the CRS of our hillshade to match the `DTM_HARV` raster. We can thus
assign the CRS of our `DTM_HARV` to our hillshade within the `projectRaster()`
function as follows: `crs = crs(DTM_HARV)`. 
Note that we are using the `projectRaster()` function on the *raster* object,
not the `data.frame()` we pass to `ggplot`.


~~~
# reproject to UTM
DTM_hill_UTMZ18N_HARV <- projectRaster(DTM_hill_HARV,
                                       crs = crs(DTM_HARV))

# compare attributes of DTM_hill_UTMZ18N to DTM_hill
crs(DTM_hill_UTMZ18N_HARV)
~~~
{: .r}



~~~
CRS arguments:
 +proj=utm +zone=18 +datum=WGS84 +units=m +no_defs +ellps=WGS84
+towgs84=0,0,0 
~~~
{: .output}



~~~
crs(DTM_hill_HARV)
~~~
{: .r}



~~~
CRS arguments:
 +proj=longlat +datum=WGS84 +no_defs +ellps=WGS84 +towgs84=0,0,0 
~~~
{: .output}



~~~
# compare attributes of DTM_hill_UTMZ18N to DTM_hill
extent(DTM_hill_UTMZ18N_HARV)
~~~
{: .r}



~~~
class       : Extent 
xmin        : 731397.3 
xmax        : 733205.3 
ymin        : 4712403 
ymax        : 4713907 
~~~
{: .output}



~~~
extent(DTM_hill_HARV)
~~~
{: .r}



~~~
class       : Extent 
xmin        : -72.18192 
xmax        : -72.16061 
ymin        : 42.52941 
ymax        : 42.54234 
~~~
{: .output}

Notice in the output above that the `crs()` of `DTM_hill_UTMZ18N_HARV` is now
UTM. However, the extent values of `DTM_hillUTMZ18N_HARV` are different from
`DTM_hill_HARV`.

> ## Challenge: Extent Change with CRS Change
> 
> Why do you think the two extents differ?
> > ## Answers
> > 
> > ~~~
> > # The extent for DTM_hill_UTMZ18N_HARV is in UTMs so the extent is in meters.
> > # The extent for DTM_hill_HARV is still in lat/long so the extent is expressed
> > # in decimal degrees.
> > ~~~
> > {: .r}
> {: .solution}
{: .challenge}

## Deal with Raster Resolution

Let's next have a look at the resolution of our reprojected hillshade.


~~~
# compare resolution
res(DTM_hill_UTMZ18N_HARV)
~~~
{: .r}



~~~
[1] 1.000 0.998
~~~
{: .output}

The output resolution of `DTM_hill_UTMZ18N_HARV` is 1 x 0.998. Yet, we know that
the resolution for the data should be 1m x 1m. We can tell `R` to force our
newly reprojected raster to be 1m x 1m resolution by adding a line of code
(`res=`).


~~~
# adjust the resolution
DTM_hill_UTMZ18N_HARV <- projectRaster(DTM_hill_HARV,
                                  crs = crs(DTM_HARV),
                                  res = 1)
# view resolution
res(DTM_hill_UTMZ18N_HARV)
~~~
{: .r}



~~~
[1] 1 1
~~~
{: .output}

Let's plot our newly reprojected raster.


~~~
# convert the reprojected raster to a data.frame()

DTM_hill_HARV_2_df <- DTM_hill_UTMZ18N_HARV  %>% 
                          rasterToPoints(., spatial = TRUE) %>% 
                          data.frame()

ggplot() +
     geom_raster(data = DTM_hill_HARV_2_df, 
                 aes(x = x, y = y, 
                   alpha = HARV_DTMhill_WGS84)
                 ) +
     geom_raster(data = DTM_HARV_df , 
             aes(x = x, y = y, 
                  fill = HARV_dtmCrop,
                  alpha=0.8)
             ) + 
     scale_fill_gradientn(name = "Elevation", colors = terrain.colors(10)) +
     guides(fill = guide_colorbar()) +
     scale_alpha(range = c(0.25, 0.65), guide = "none") +
    theme_bw() +
    theme(panel.grid.major = element_blank(), 
          panel.grid.minor = element_blank()) +
     theme(axis.title.x = element_blank(),
           axis.title.y = element_blank()) +
     ggtitle("Digital Terrain Model - NEON Harvard Forest Field Site") +
     coord_equal()
~~~
{: .r}

<img src="../fig/rmd-03-plot-projected-raster-1.png" title="plot of chunk plot-projected-raster" alt="plot of chunk plot-projected-raster" style="display: block; margin: auto;" />

We have now successfully draped the Digital Terrain Model on top of our
hillshade to produce a nice looking, textured map!

> ## Challenge: Reproject, then Plot a Digital Terrain Model
> Create a map of the
> <a href="http://www.neoninc.org/science-design/field-sites/san-joaquin-experimental-range" target="_blank" >San Joaquin Experimental Range</a>
field site using the `SJER_DSMhill_WGS84.tif` and `SJER_dsmCrop.tif` files.
> 
> Reproject the data as necessary to make things line up!
> > ## Answers
> > 
> > ~~~
> > # import DTM
> > DSM_SJER <- raster("data/NEON-DS-Airborne-Remote-Sensing/SJER/DSM/SJER_dsmCrop.tif")
> > # import DTM hillshade
> > DSM_hill_SJER_WGS <-
> > raster("data/NEON-DS-Airborne-Remote-Sensing/SJER/DSM/SJER_DSMhill_WGS84.tif")
> > 
> > # reproject raster
> > DTM_hill_UTMZ18N_SJER <- projectRaster(DSM_hill_SJER_WGS,
> >                                   crs = crs(DSM_SJER),
> >                                   res = 1)
> > 
> > # convert to data.frames
> > DSM_SJER_df <- DSM_SJER %>% 
> >   rasterToPoints(., spatial = TRUE) %>% 
> >   data.frame()
> > 
> > DSM_hill_SJER_df <- DTM_hill_UTMZ18N_SJER  %>% 
> >   rasterToPoints(., spatial = TRUE) %>% 
> >   data.frame()
> > 
> > ggplot() +
> >      geom_raster(data = DSM_hill_SJER_df, 
> >                  aes(x = x, y = y, 
> >                    alpha = SJER_DSMhill_WGS84)
> >                  ) +
> >      geom_raster(data = DSM_SJER_df, 
> >              aes(x = x, y = y, 
> >                   fill = SJER_dsmCrop,
> >                   alpha=0.8)
> >              ) + 
> >      scale_fill_gradientn(name = "Elevation", colors = terrain.colors(10)) +
> >      guides(fill = guide_colorbar()) +
> >      scale_alpha(range = c(0.25, 0.65), guide = "none") +
> >     theme_bw() +
> >     theme(panel.grid.major = element_blank(), 
> >           panel.grid.minor = element_blank()) +
> >      theme(axis.title.x = element_blank(),
> >            axis.title.y = element_blank()) +
> >      ggtitle("DSM with Hillshade - NEON SJER Field Site") +
> >      coord_equal()
> > ~~~
> > {: .r}
> > 
> > <img src="../fig/rmd-03-challenge-code-reprojection-1.png" title="plot of chunk challenge-code-reprojection" alt="plot of chunk challenge-code-reprojection" style="display: block; margin: auto;" />
> {: .solution}
{: .challenge}

If you completed the San Joaquin plotting challenge in the
[Plot Raster Data in R]({{ site.baseurl }}/R/Plot-Rasters-In-R#challenge-create-dtm--dsm-for-sjer)
tutorial, how does the map you just created compare to that map?


