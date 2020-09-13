---
title: "Landsliding Into PostGIS With KML Files"
date: "2011-01-05T09:25:00-08:00"
template: "post"
draft: false
slug: "landsliding-into-postgis-with-kml-files"
category: "Geospatial"
tags:
  - esri shapefile
  - gdal
  - geo
  - geographic
  - geospatial
  - gis
  - homebrew
  - kml
  - kmz
  - landslides
  - latitude
  - libkml
  - longitude
  - macosx
  - ogr2ogr
  - postgis
  - postgresql
  - shapefile
  - shp
  - sql
description: "In this post I will show, in repeatable steps, how to install PostGIS, load in geospatial data found in a KML file and run queries against that data. The focus of this geospatial data will be landslides and our resulting database will allow us to query, using longitude and latitude co-ordinates, the landslide status of a specific geographical point."
socialImage:
  publicURL: "/media/images/2011/01/postgis_kml_sq.jpg"
---
<a href="https://www.flickr.com/photos/hitchster/2871036863/" title="Landslide damaged roadway (2005) by Hitchster, on Flickr"><img align="left" alt="Landslide damaged roadway (2005)" height="313" src="/media/images/2011/01/postgis_kml.jpg" style="border: none; margin-right: 20px" width="235"/></a>

In this post I will show, in repeatable steps, how to install [PostGIS](https://postgis.refractions.net/), load in geospatial data found in a [KML](https://en.wikipedia.org/wiki/Keyhole_Markup_Language) file and run queries against that data. The focus of this geospatial data will be landslides and our resulting database will allow us to query, using longitude and latitude co-ordinates, the landslide status of a specific geographical point. The returned status for the given co-ordinates will be “Mostly Landslides”, “Few Landslides”, “Flat Land”, “Not Mapped”.

## KML / KMZ Files

KML files define, in XML, geographical data in three dimensions. A _KMZ_ file is simply a zipped KML file.

You can find many weird and wonderful KML files on the Internet, either via [Google Search](https://www.google.ca/search?q=kml+files) or via [Google Earth Gallery](https://www.google.com/gadgets/directory?synd=earth&amp;cat=featured&amp;preview=on).

>  
> __Keyhole Markup Language (KML)__ is an XML schema for expressing geographic annotation and visualization within Internet-based, two-dimensional maps and three-dimensional Earth browsers. KML was developed for use with Google Earth, which was originally named Keyhole Earth Viewer. It was created by Keyhole, Inc, which was acquired by Google in 2004. The name “Keyhole” is an homage to the KH reconnaissance satellites, the original eye-in-the-sky military reconnaissance system first launched in 1976. KML is an international standard of the Open Geospatial Consortium. Google Earth was the first program able to view and graphically edit KML files.  
>  – [KML Wikipedia](https://en.wikipedia.org/wiki/Keyhole_Markup_Language)
> 

## PostGIS

[PostGIS](https://postgis.refractions.net/) is the number one architecture for dealing with geospatial data within a relational database. It works within PostgreSQL.

While working with [netSIGN](https://www.netsign.com) I had the opportunity to work briefly with PostGIS, which provides support for geographic objects within PostgreSQL. In that instance I was precisely identifying real-estate locations from longitude and latitude co-ordinates using [The City Of Vancouver’s Area Boundary KML file](https://data.vancouver.ca/datacatalogue/localAreaBoundary.htm). This time we will be looking at [San Francisco Bay-Area Landslides](https://earthquake.usgs.gov/regional/nca/bayarea/landslides.php).

### Installing PostGIS

On Mac OS X you can install both PostGIS, and it’s dependency PostgreSQL, with [Homebrew](/homebrew-intro-to-the-mac-os-x-package-installer). For simplicity we assume that you already have PostgreSQL installed. 

```
sudo brew install postgis
```

### Configuring PostGIS

Login as the _postgres_ user.

```
sudo su postgres
```

### Create A PostGIS Template Database

Now we will create a template database for PostGIS within PostgreSQL from which we can create our own PostGIS database.

```bash
TEMPLATE_DB_NAME=template_postgis

# Create the template spatial database.
createdb -E UTF8 $TEMPLATE_DB_NAME

# Add PLPGSQL language support.
createlang -d $TEMPLATE_DB_NAME plpgsql

# Load the PostGIS extensions
for SQL_NAME in postgis postgis_comments spatial_ref_sys
do
    psql -d $TEMPLATE_DB_NAME -f /usr/local/Cellar/postgis/1.5.2/share/postgis/$SQL_NAME.sql
done

# Finalize our PostGIS template database
cat <<EOS | psql -d $TEMPLATE_DB_NAME
UPDATE pg_database SET datistemplate = TRUE WHERE datname = '$TEMPLATE_DB_NAME';
REVOKE ALL ON SCHEMA public FROM public;
GRANT USAGE ON SCHEMA public TO public;
GRANT ALL ON SCHEMA public TO postgres;
GRANT SELECT, UPDATE, INSERT, DELETE ON TABLE public.geometry_columns TO PUBLIC;
GRANT SELECT, UPDATE, INSERT, DELETE ON TABLE public.spatial_ref_sys TO PUBLIC;
VACUUM FULL FREEZE;
EOS
```

### Create The Database From The Template

```bash
DB_NAME="landslide_db"
DB_USER="landslide_user"

createuser --no-superuser --createdb --no-createrole $DB_USER
createdb --template=template_postgis --encoding=UTF8 --owner=$DB_USER $DB_NAME
echo "GRANT ALL ON SCHEMA public TO $DB_USER" | psql $DB_NAME
for t in geography_columns geometry_columns spatial_ref_sys
do
    echo "ALTER TABLE $t OWNER TO $DB_USER" | psql -d $DB_NAME
done

```

## Preparing The Geospatial Data For PostGIS

I will use some test data from [San Francisco Bay-Area Landslides](https://earthquake.usgs.gov/regional/nca/bayarea/landslides.php). From [earthquake.usgs.gov](https://earthquake.usgs.gov) we can download a [KMZ file](https://earthquake.usgs.gov/regional/nca/bayarea/kml/landslides.kmz) which contains the geographic information, in the form of 2-D polygons, of the landslide areas.

```bash
wget https://earthquake.usgs.gov/regional/nca/bayarea/kml/landslides.kmz
```

### Converting KML File To ESRI Shapefile Using Ogr2ogr

We need to install the _ogr2ogr_ command-line conversion tool. This is included with [gdal](https://www.gdal.org/) and [FWTools](https://fwtools.maptools.org/). If you install _gdal-bin_ on Linux via _apt-get_ then you will probably get an older version that does not decode KML 2.2 files (required in this example). I recommend building [gdal-bin from source](https://trac.osgeo.org/gdal/wiki/DownloadSource) on Linux.

On my Mac I will install gdal, and hence ogr2ogr, using [Homebrew](/homebrew-intro-to-the-mac-os-x-package-installer).

We first have to install [libkml](https://code.google.com/p/libkml/), which is dependency of gdal if we want read KML 2.2 files. libkml is a KML file parsing engine written by Google and is used in Google Earth and Google Maps. Ogr2ogr can use libkml to read the KML files. It does have it’s own KML parser, but that will not work with the KML 2.2 file we will try to convert and generally is not as good as libkml at reading KML files. Usually, I would install libkml using [Homebrew](/homebrew-intro-to-the-mac-os-x-package-installer), but I found that you can only install libkml 1.2 and gdal 1.8.0beta1 requires libkml 1.3. Therefore, I will install libkml from the libkml source respository, which is currently only way to get libkml 1.3.

```bash
sudo brew install libkml # only version 1.2

svn checkout https://libkml.googlecode.com/svn/trunk libkml # currently version 1.3
cd libkml
./autogen.sh
./configure
make
sudo make install
```

That’s libkml installed. Now, let’s use [Homebrew](/homebrew-intro-to-the-mac-os-x-package-installer) to install the latest version of gdal.

```bash
sudo brew install gdal --HEAD --complete
```

I use _–complete_ to install all the additional file format parsers.

_–HEAD_ is used to download the latest version from the gdal Subversion repository. I use –HEAD because, as of writing this, the current stable version is 1.7.3 and it does not provide libkml support, which is needed to read KML 2.2 files. The current version in Subversion is 1.8.0beta1.

Now the _ogr2ogr_ conversion utility should be installed. KMZ is a zipped KML file. It is just a regular zip file, so we can unzip this by simply using the unzip command. libkml does recognize the kmz extension, so we would not usually need to do this step, but landslides.kmz does not contain the data we want, only references to it via several [NetworkLinks](https://code.google.com/apis/kml/documentation/kml_tut.html#network_links) to other KMZ files. 

```bash
unzip landslides.kmz
```

<small>If you have problems unzip a .kmz file, then simply rename to have a .zip extension</small>

When we unzip landslides.kmz, we find it contains only one file called _doc.kml_. Here is a what the doc.kml looks like…

```xml
<?xml version="1.0" encoding="UTF-8"?>
<kml xmlns="https://www.opengis.net/kml/2.2" xmlns:gx="https://www.google.com/kml/ext/2.2" xmlns:kml="https://www.opengis.net/kml/2.2" xmlns:atom="https://www.w3.org/2005/Atom">
<Document>
  <name>Landslides</name>
  <LookAt>
    <longitude>-122.2499999599815</longitude>
    <latitude>37.62500001065743</latitude>
    <altitude>0</altitude>
    <range>413167.7811227545</range>
    <tilt>0</tilt>
    <heading>2.443095370245979e-008</heading>
    <altitudeMode>relativeToGround</altitudeMode>
  </LookAt>
  <Style>
    <ListStyle>
      <listItemType>checkHideChildren</listItemType>
      <bgColor>00ffffff</bgColor>
      <maxSnippetLines>2</maxSnippetLines>
    </ListStyle>
  </Style>
  <NetworkLink id="1">
    <name>Aetna_Springs</name>
    <Region>
      <LatLonAltBox>
        <north>38.75</north>
        <south>38.625</south>
        <east>-122.375</east>
        <west>-122.5</west>
        <minAltitude>0</minAltitude>
        <maxAltitude>0</maxAltitude>
      </LatLonAltBox>
      <Lod>
        <minLodPixels>450</minLodPixels>
        <maxLodPixels>-1</maxLodPixels>
        <minFadeExtent>128</minFadeExtent>
        <maxFadeExtent>128</maxFadeExtent>
      </Lod>
    </Region>
    <Link>
      <href>https://earthquake.usgs.gov/regional/nca/bayarea/kml/Landslides/Landslides_Aetna_Springs.kmz</href>
      <ViewRefreshMode>onRegion</ViewRefreshMode>
    </Link>
  </NetworkLink>
  <NetworkLink id="2">
    <name>Allendale</name>
    <Region>
      <LatLonAltBox>
        <north>38.5</north>
        <south>38.375</south>
        <east>-121.875</east>
        <west>-122</west>
        <minAltitude>0</minAltitude>
        <maxAltitude>0</maxAltitude>
      </LatLonAltBox>
      <Lod>
        <minLodPixels>450</minLodPixels>
        <maxLodPixels>-1</maxLodPixels>
        <minFadeExtent>128</minFadeExtent>
        <maxFadeExtent>128</maxFadeExtent>
      </Lod>
    </Region>
    <Link>
      <href>https://earthquake.usgs.gov/regional/nca/bayarea/kml/Landslides/Landslides_Allendale.kmz</href>
      <ViewRefreshMode>onRegion</ViewRefreshMode>
    </Link>
  </NetworkLink>
  <NetworkLink id="3">
    <name>Altamont</name>
    <Region>
      <LatLonAltBox>
        <north>37.75</north>
        <south>37.625</south>

```



Yes, KML is XML.

You will notice several &lt;NetworkLink&gt; blocks that contain a &lt;Link&gt; block that contains &lt;href&gt; tags that contain a web link to another KMZ file. Ideally these would be sucked in libkml when we parse the KML/KMZ file, but as Google says in it’s documentation…

>  
> In their simplest form, network links are a useful way to split one large KML file into smaller, more manageable files on the same computer.
> 
> Despite the name, a &lt;NetworkLink&gt; does not necessarily load files from the network.
> 
> <small>[ – Courtesy of Google’s KML documentation](https://code.google.com/apis/kml/documentation/kml_tut.html#network_links)</small>
> 

For this example, I’m just going to grab the first of these referenced KMZ files, [Landslides\_Aetna\_Springs.kmz](https://earthquake.usgs.gov/regional/nca/bayarea/kml/Landslides/Landslides_Aetna_Springs.kmz).

We will first convert this KMZ into a ESRI Shapefile using ogr2ogr.

```bash
ogr2ogr -f "ESRI Shapefile" landslides.shp Landslides_Aetna_Springs.kmz
```

Although, we got a few warnings, we did get our shapefile (actually, several files).

```
Warning 6: Normalized/laundered field name: 'description' to 'descriptio'
Warning 6: Field timestamp create as date field, though DateTime requested.

Warning 6: Field begin create as date field, though DateTime requested.

Warning 6: Field end create as date field, though DateTime requested.

Warning 6: Normalized/laundered field name: 'altitudeMode' to 'altitudeMo'

```

The ESRI Shapefile comprises of these files, although we usually only need to reference the .shp file.

```
landslides.shp
landslides.shx
landslides.prj
landslides.dbf

```

### Converting ESRI Shapefile To SQL Using Shp2pgsql

Since PostGIS runs inside PostgreSQL, we are going need some SQL, so I will convert that ESRI Shapefile to SQL using the _shp2pgsql_ command-line tool. The _shp2pgsql_ tool comes with the PostGIS install, so you can run this from anywhere that you have installed PostGIS.

```
shp2pgsql -s 4326 -I -c -W UTF-8 landslides.shp landslides > landslides.sql
```

This generates _landslide.sql_ which contains the geolocation data from Landslides\_Aetna\_Springs.kmz as SQL statements.

-s specifies the SRID field. Since Google Maps uses SRID 4326 (“WGS84″) for this, we will use the same.

>  
> A Spatial Reference System Identifier (SRID) is a unique value used to unambiguously identify projected, unprojected, and local spatial coordinate system definitions. These coordinate systems form the heart of all GIS applications.  
> <small>[ – Wikipedia](https://en.wikipedia.org/wiki/SRID)</small>
> 

We also specifiy _landslides_ as the name of database table that we will be inserting into. landslides.sql will contain a create table statement using this name and the subsequent INSERT statements within landslides.sql will insert into this table.

Here are the other options I have used for the _shp2pgsql_ command.

<table>
<tr>
<td style="width: 30px">-c</td>
<td>Creates a new table and populates it, this is the default if you do not specify any options.</td>
</tr>
<tr>
<td>-I</td>
<td>Create a spatial index on the geocolumn.</td>
</tr>
<tr>
<td>-S</td>
<td>Generate simple geometries instead of MULTI geometries.</td>
</tr><tr>
<td>-W</td>
<td>&lt;encoding&gt; Specify the character encoding of Shape’s attribute column. (default : “WINDOWS-1252″)</td>
</tr>
</table>



### Multipolygons

Previously I have also used the _-S_ switch with shp2pgsql, but for this KMZ file I got the following message.

```
We have a Multipolygon with 40 parts, can't use -S switch!
```

<table>
<tr>
<td>-S</td>
<td>Generate simple geometries instead of MULTI geometries.</td>
</tr><tr>
</tr></table>

## The SQL

Let’s take a look at the SQL generated…

```sql
SET CLIENT_ENCODING TO UTF8;
SET STANDARD_CONFORMING_STRINGS TO ON;
BEGIN;
CREATE TABLE "landslides" (gid serial PRIMARY KEY,
"name" varchar(80),
"descriptio" varchar(80),
"timestamp" date,
"begin" date,
"end" date,
"altitudemo" varchar(80),
"tessellate" numeric(10,0),
"extrude" numeric(10,0),
"visibility" numeric(10,0));
SELECT AddGeometryColumn('','landslides','the_geom','4326','MULTIPOLYGON',4);

INSERT INTO "landslides" ("name","descriptio","timestamp","begin","end","altitudemo",
"tessellate","extrude","visibility",the_geom) VALUES ('Mostly Landslide','Mostly Landslide',
NULL,NULL,NULL,NULL,'1','0','-1',
'01060000E0E61000002800000001030000E0E610000001000000060000009B559FABAD985EC0000000000060434000000
0000000F03F0000000000000000C0EC9E3C2C985EC00000000000604340000000000000F03F0000000000000000711B0DE
02D985EC032207BBDFB5F4340000000000000F03F0000000000000000386744696F985EC032207BBDFB5F4340000000000
000F03F000000000000000038F8C264AA985EC032207BBDFB5F4340000000000000F03F00000000000000009B559FABAD9
85EC00000000000604340000000000000F03F000000000000000001030000E0E61000000100000005000000613255302A9
95EC00000000000604340000000000000F03F00000000000000004CA60A4625995EC00000000000604340000000000000
F03F0000000000000000FED478E926995EC0DC291DACFF5F4340000000000000F03F0000000000000000B003E78C28995E
C00000000000604340000000000000F03F0000000000000000613255302A995EC00000000000604340000000000000F03F
0000000... (truncated)

INSERT INTO "landslides" ("name","descriptio","timestamp","begin","end","altitudemo",
"tessellate","extrude","visibility",the_geom) VALUES ('Few Landslides','Few Landslides',
NULL,NULL,NULL,NULL,'1','0','-1',
'01060000E0E61000001B00000001030000E0E610000004000000490600000000000000985EC0A83AE466B85D434000000
0000000F03F0000000000000000C7BAB88D06985EC068B3EA73B55D4340000000000000F03F000000000000000040A4DF
BE0E985EC0282CF180B25D4340000000000000F03F0000000000000000075F984C15985EC02F51BD35B05D434000000000
0000F03F00000000000000006ABC749318985EC076FD82DDB05D4340000000000000F03F00000000000000001CEBE2361
A985EC004560E2DB25D4340000000000000F03F0000000000000000CE1951DA1B985EC0D95A5F24B45D43400000000000
00F03F000000000000000095D4096822985EC0761A69A9BC5D4340000000000000F03F00000000000000004703780B249
85EC02849D74CBE5D4340000000000000F03F00000000000000005C8FC2F528985EC092CB7F48BF5D4340000000000000F
03F0000000000000000C0EC9E3C2C985EC0B6A1629CBF5D4340000000000000F03F0000000000000000234A7B832F985EC
092CB7F48BF5D4... (truncated)

```



The INSERT commands are pretty long, so I’ve truncated them here, but you can get the general idea.

## Loading SQL Into PostGIS

We can now load the SQL we generated into our database. This is an easy step.

```bash
psql -d $DB_NAME $DB_USER < landslides.sql
```

You see something like this, if all goes well.

```sql
SET
SET
BEGIN
NOTICE:  CREATE TABLE will create implicit sequence "landslides_gid_seq" for serial column "landslides.gid"
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "landslides_pkey" for table "landslides"
CREATE TABLE
                       addgeometrycolumn
----------------------------------------------------------------
 public.landslides.the_geom SRID:4326 TYPE:MULTIPOLYGON DIMS:4
(1 row)

INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
CREATE INDEX
COMMIT
```

## Running SQL Queries Against Our PostGIS Database

```bash
DB_NAME="landslide_db"
DB_USER="landslide_user"

psql -d $DB_NAME $DB_USER
```

Our landslide database has four definitions of landslides, each mapping to a complex set of polgons (table column “the\_geom” is TYPE:MULTIPOLYGON). Here are their names.

```sql
landslide_db=> SELECT name FROM landslides;
       name
------------------
 Mostly Landslide
 Few Landslides
 Flat Land
 Not Mapped
(4 rows)
```

Let’s look for landslides at latitude and longtitude position -122.3800, 38.7499. Here we find a bad location (meaning you would not build your house there) since this is in the bounds of “Mostly Landslide”.

```sql
landslide_db=> SELECT name FROM landslides WHERE ST_Contains(the_geom,
ST_GeomFromWKB(ST_AsEWKB('POINT(-122.3800 38.7499)'::geometry), 4326));
       name
------------------
 Mostly Landslide
(1 row)
```

Let’s find somewhere with less landslides. Maybe -122.3752, 38.7321?

```sql
landslide_db=> SELECT name FROM landslides WHERE ST_Contains(the_geom,
ST_GeomFromWKB(ST_AsEWKB('POINT(-122.3752 38.73215)'::geometry), 4326));
      name
----------------
 Few Landslides
(1 row)
```

That is pretty good, but flat land would be better.

```sql
landslide_db=> SELECT name FROM landslides WHERE ST_Contains(the_geom,
ST_GeomFromWKB(ST_AsEWKB('POINT(-122.4015 38.74941)'::geometry), 4326));
   name
-----------
 Flat Land
(1 row)
```

Bingo! We’ve found some flat land that is landslide free. This seems like a safe place to conclude this blog post.

## Conclusion

In this blog post I went through the steps needed to convert and load a KML 2.2 file into a PostGIS enabled PostgreSQL database. We installed PostGIS and all the tools needed to convert our KML file into an ESRI Shapefile and then into SQL statements that could be ran in PostgreSQL. We can now run SQL queries against our database which will return the landslide status for any given longitude and latitude within the focus area.
