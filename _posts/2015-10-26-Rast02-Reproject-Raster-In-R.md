---
layout: post
title: "Lesson 02: When Rasters Don't Line Up - Reproject Raster Data in R"
date:   2015-10-27
authors: [Jason Williams, Jeff Hollister, Kristina Riemer, Mike Smorul, Zack Brym, Leah Wasser]
contributors: [Megan A. Jones]
packagesLibraries: [raster, rgdal]
dateCreated:  2015-10-23
lastModified: 2015-12-17
category: spatio-temporal-workshop
tags: [raster-ts-wrksp, raster]
mainTag: raster-ts-wrksp
description: "This lesson explains how to reproject a raster in `R` using the
`projectRaster()` function in the raster package."
code1: SR02-Reproject-Raster-In-R.R
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: /R/Reproject-Raster-In-R
comments: false
---

{% include _toc.html %}

##About

It is common for us to encounter raster data layers that do not "line up".
Rasters that don't line up are most often in different Coordinate Reference
Systems (CRS). This can cause problems with plotting and analyzing raster data.

This lesson explains how to deal with rasters in different, known CRSs. It will
walk though reprojecting rasters in `R` using the `projectRaster()` function in
the raster package.

**R Skill Level:** Intermediate - you've got the basics of `R` down.

<div id="objectives" markdown="1">

###Goals / Objectives

After completing this activity, you will:

* Be able to reproject a raster in R

###Challenge Code
Throughout the lesson we have Challenges that reinforce learned skills. Possible
solutions to the challenges are not posted on this page, however, the code for 
each challenge is in the `R` code that can be downloaded for this lesson (see 
footer on this page).

###Things You'll Need To Complete This Lesson

Please be sure you have the most current version of `R` and, preferably,
RStudio to write your code.

###R Libraries to Install:

* **raster:** `install.packages("raster")`
* **rgdal:** `install.packages("rgdal")`

####Data to Download

<a href="https://ndownloader.figshare.com/files/3579867" class="btn btn-success"> Download NEON Airborne Observation Platform Raster Data Teaching Subset</a> 

The LiDAR and imagery data used to create this raster teaching data subset were
collected over the NEON <a href="http://www.neoninc.org/science-design/field-sites/harvard-forest" target="_blank" >Harvard Forest</a>
and 
<a href="http://www.neoninc.org/science-design/field-sites/san-joaquin-experimental-range" target="_blank" >San Joaquin Experimental Range</a>
field sites and processed at
<a href="http://www.neoninc.org" target="_blank" >NEON </a> 
headquarters. The entire dataset can be accessed by request from the 
<a href="http://www.neoninc.org/data-resources/get-data/airborne-data" target="_blank"> NEON Airborne Data Request Page on the NEON Website.</a>

####Setting the Working Directory
The code in this lesson assumes that you have set your working directory to the
location of the unzipped file of data downloaded above.  If you would like a
refresher on setting the working directory, please view the [Setting A Working Directory In R]({{site.baseurl}}/R/Set-Working-Directory "R Working Directory Lesson") 
lesson prior to beginning this lesson.

###Raster Lesson Series 
This lesson is a part of a series of raster data in R lessons:

* [Lesson 00 - Intro to Raster Data in R]({{ site.baseurl}}/R/Introduction-to-Raster-Data-In-R/)
* [Lesson 01 - Plot Raster Data in R]({{ site.baseurl}}/R/Plot-Rasters-In-R/)
* [Lesson 02 - Reproject Raster Data in R]({{ site.baseurl}}/R/Reproject-Raster-In-R/)
* [Lesson 03 - Raster Calculations in R]({{ site.baseurl}}/R/Raster-Calculations-In-R/)
* [Lesson 04 - Work With Multi-Band Rasters - Images in R]({{ site.baseurl}}/R/Multi-Band-Rasters-In-R/)
* [Lesson 05 - Raster Time Series Data in R]({{ site.baseurl}}/R/Raster-Times-Series-Data-In-R/)
* [Lesson 06 - Plot Raster Time Series Data in R Using RasterVis and LevelPlot]({{ site.baseurl}}/R/Plot-Raster-Times-Series-Data-In-R/)
* [Lesson 07- Extract NDVI Summary Values from a Raster Time Series]({{ site.baseurl}}/R/Extract-NDVI-From-Rasters-In-R/)

###Additional Resources

* <a href="http://cran.r-project.org/web/packages/raster/raster.pdf" target="_blank">
Read more about the `raster` package in `R`.</a>

</div>

#Raster Projection in R

In the [Plot Raster Data in R]({{ site.baseurl}}/R/Plot-Rasters-In-R/) 
lesson, we learned how to layer a raster file on top of a hillshade for a nice
looking basemap.
This worked well when all of our data were cleaned up for us in advance. But 
what happens when things don't line up? Let's have a look.

We will use the `raster` and `rgdal` packages in this lesson.  


    #load raster package
    library(raster)
    library(rgdal)

Let's create an layered map of the Harvard Forest Digital Terrain Model 
(`DTM_HARV`) with the hillshade (`DTM_hill_HARV`) used as a base layer.


    #import DTM
    DTM_HARV <- raster("NEON_RemoteSensing/HARV/DTM/HARV_dtmcrop.tif")
    #import DTM hillshade
    DTM_hill_HARV <- raster("NEON_RemoteSensing/HARV/DTM/HARV_DTMhill_WGS84.tif")
    
    #plot hillshade using a grayscale color ramp 
    plot(DTM_hill_HARV,
        col=grey(1:100/100),
        legend=F,
        main="DTM Hillshade\n NEON Harvard Forest")
    
    #overlay the DTM on top of the hillshade
    plot(DTM_HARV,
         col=terrain.colors(10),
         alpha=0.4,
         add=T,
         legend=F)

![ ]({{ site.baseurl }}/images/rfigs/02-Reproject-Raster-In-R/import-DTM-hillshade-1.png) 

Our results are curious - the Digital Terrain Model (`DTM_HARV`) did not plot on
top of our hillshade. The hillshade plotted just fine on it's own. Let's try to 
plot the DTM on it's own to make sure there are data there.


    #Plot DTM 
    plot(DTM_HARV,
         col=terrain.colors(10),
         alpha=1,
         legend=F,
         main="Digital Terrain Model\n NEON Harvard Forest")

![ ]({{ site.baseurl }}/images/rfigs/02-Reproject-Raster-In-R/plot-DTM-1.png) 

It appears as if our DTM contains data and plots just fine. The layers likely
do not line up. A likely culprit for layers not lining up is the Coordinate 
Reference System (CRS). Let's explore our data.


    #view crs for DTM
    crs(DTM_HARV)

    ## CRS arguments:
    ##  +proj=utm +zone=18 +datum=WGS84 +units=m +no_defs +ellps=WGS84
    ## +towgs84=0,0,0

    #view crs for hillshade
    crs(DTM_hill_HARV)

    ## CRS arguments:
    ##  +proj=longlat +datum=WGS84 +no_defs +ellps=WGS84 +towgs84=0,0,0

Aha! `DTM_HARV` is in the UTM projection. `DTM_hill_HARV` is in latitude and
longitude, a non-projected, geographic CRS. Because the two rasters are in
different CRSs, they won't line up. We will need to *reproject* DTM_HARV into
the UTM CRS (or reproject the hillshade into lat/long). 

#Reproject Rasters
When things don't line up, it is often due to differences in CRS. In this case,
our DTM is in UTM zone 18. However, our hillshade is in the geographic 
coordinate system (latitude and longitude). 

We can use the `projectRaster` function to reproject a raster into a new `CRS`.
Keep in mind that reprojection only works when you first have a *defined* `CRS`
for the raster object that you want to reproject. It cannot be used if *no*
`CRS` is defined. In this case, the `DTM_hill_HARV` does have a defined `CRS`. 

When using the `projectRaster()` function, you need to define two key things:

1. the object you want to reproject and 
2. the CRS that you want to reproject it to. 

The syntax is `projectRaster(RasterObject,crs=CRSToReprojectTo)`

Since we want the `CRS` of our hillshade to match the `DTM_HARV` raster we can 
tell `R` to use the `CRS` from `DTM_HARV` as the `CRS` for this reprojetion.


    #reproject to UTM
    DTM_hill_UTMZ18N_HARV <- projectRaster(DTM_hill_HARV, 
                                           crs=crs(DTM_HARV))
    
    #compare attributes of DTM_hill_UTMZ18N to DTM_hill
    crs(DTM_hill_UTMZ18N_HARV)

    ## CRS arguments:
    ##  +proj=utm +zone=18 +datum=WGS84 +units=m +no_defs +ellps=WGS84
    ## +towgs84=0,0,0

    crs(DTM_hill_HARV)

    ## CRS arguments:
    ##  +proj=longlat +datum=WGS84 +no_defs +ellps=WGS84 +towgs84=0,0,0

    #compare attributes of DTM_hill_UTMZ18N to DTM_hill
    extent(DTM_hill_UTMZ18N_HARV)

    ## class       : Extent 
    ## xmin        : 731397.3 
    ## xmax        : 733205.3 
    ## ymin        : 4712403 
    ## ymax        : 4713907

    extent(DTM_hill_HARV)

    ## class       : Extent 
    ## xmin        : -72.18192 
    ## xmax        : -72.16061 
    ## ymin        : 42.52941 
    ## ymax        : 42.54234

Notice in the output above that the CRS of `DTM_hillUTMZ18N_HARV` is now UTM. 
However, the extent of the object is different. 

Why do you think this is?  

Note: When you are reprojecting a raster, you are moving it from one "grid" 
to another. Thus you are modifying the data! Keep this in mind as you work with
raster data. {: .notice2}


##Dealing with Raster Resolution

Let's next have a look at the resolution of the two rasters.  


    #compare resolution
    res(DTM_hill_HARV)

    ## [1] 1.22e-05 8.99e-06

    res(DTM_hill_UTMZ18N_HARV)

    ## [1] 1.000 0.998

The output resolution of `DTM_hill_UTMZ18N_HARV` is 1 x 0.998. Yet, we know that
the resolution for the data should be 1m x 1m. We can tell `R` to force our newly
reprojected raster to be 1m resolution by adding a line of code (`res=`).  


    #adjust the resolution 
    DTM_hill_UTMZ18N_HARV <- projectRaster(DTM_hill_HARV, 
                                      crs=crs(DTM_HARV),
                                      res=1)
    #view resolution
    res(DTM_hill_UTMZ18N_HARV)

    ## [1] 1 1


Once we have reprojected the raster, we can try to plot again!


    #plot newly reprojected hillshade
    plot(DTM_hill_UTMZ18N_HARV,
        col=grey(1:100/100),
        legend=F,
        main="DTM with Hillshade\n NEON Harvard Forest Field Site")
    
    #overlay the DTM on top of the hillshade
    plot(DTM_HARV,
         col=rainbow(100),
         alpha=0.4,
         add=T,
         legend=F)

![ ]({{ site.baseurl }}/images/rfigs/02-Reproject-Raster-In-R/plot-projected-raster-1.png) 

We have now successfully layered the Digital Terrain Model on top of our
hillshade to produce a nice looking, textured base map! 

##Challenge: Reproject, then Plot a Digital Terrain Model 
Create a map of the <a href="http://www.neoninc.org/science-design/field-sites/san-joaquin-experimental-range" target="_blank" >San Joaquin Experimental Range</a>
field site using the `SJER_DSMhill_WGS84.tif` and `SJER_dsmCrop.tif` files. 

Reproject the data as necessary to make things line up!

![ ]({{ site.baseurl }}/images/rfigs/02-Reproject-Raster-In-R/challenge-code-reprojection-1.png) 

If you completed the San Joaquin plotting challenge in the
[Plot Raster Data in R]({{ site.baseurl}}/R/Plot-Rasters-In-R/) 
lesson, how does the map you just created compare to that map? 

