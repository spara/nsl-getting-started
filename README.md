# Getting Started with NearSpace Labs API and Anaconda

The [`nsl.stac`](https://pypi.org/project/nsl.stac/) Python package lets you find aerial imagery by area of interest, data, and other attributes. It also downloads Near Space Labs Geotiffs and thumbnails. The client connects to Near Space Labs [STAC](https://stacspec.org/) service for imagery metadata queries and imagery server to retrieve imagery.

## What you need for this tutorial

Request access to the NearSpace Labs [API](https://nearspacelabs.com/#nearspacelabs). You need these credentials to access services.

For this tutorial, install [Anaconda](https://www.anaconda.com/products/individual) which is a Python distribution that includes [JupyterLabs](https://jupyter.org/install). JupyterLabs provides a web-based interactive environment for data science and machine learning.

We will also use [`nsl.stac`](https://pypi.org/project/nsl.stac/) to connect to NearSpace Labs STAC server and retrieve imagery.

### Best practice

For each Python project, it's best practice to create a virtual environment that contains the packages required by the project. Anaconda includes a package manager called  [`conda`](https://docs.conda.io/en/latest/). Run the following to create a new virtual `conda` environment and activate it.

```bash
$ conda create --name venv python=3.8
$ conda activate venv
```

### Install packages

We need to add the JupyterLabs and numpy to our environment. Note that `nsl.stac` requires `numpy` 1.17.3 or greater and we will install it in the conda environment before installing `nsl.stac`.

```bash
$ conda install jupyter
$ conda install numpy=1.17.3
$ conda install matplotlib
$ conda install -c conda-forge rasterio
```

### Adding the nsl.stac package

The conda package manager uses the [Anaconda](https://repo.anaconda.com/) or [CondaForge](https://conda-forge.org/) repositories. The `nsl.stac` package is not available in those repositories, but we can still use pip to install it from the [PyPi](https://pypi.org/) repository. 

```bash

$ pip install nsl.stac
```

### Starting Jupyter

We can start a Jupyter server with your NearSpace Labs credentials

```
$ NSL_ID="<my_id>" NSL_SECRET="my_secret" jupyter notebook
```

![notebook](./img/jupyter_notebook.png)

## For the impatient

Open a new notebook.

Copy and paste the code example into the notebook to download and view a geotiff.

> This example is modified from https://github.com/nearspacelabs/stac-client-python#first-code-example

<br>
<br>

<details><summary>Query and view image code</summary>

```python
import tempfile, os
from IPython.display import Image, display
from datetime import date
import rasterio
from rasterio.plot import show
from nsl.stac import StacRequest, GeometryData, ProjectionData
from nsl.stac import enum, utils
from nsl.stac.client import NSLClient

# the client package stubs out a little bit of the gRPC connection code 
# get a client interface to the gRPC channel. This client singleton is threadsafe
client = NSLClient()

# our area of interest will be the coordinates of the UT Stadium in Austin, Texas
# the order of coordinates here is longitude then latitude (x, y). The results of our query 
# will be returned only if they intersect this point geometry we've defined (other geometry 
# types besides points are supported)
# This string format, POINT(float, float) is the well-known-text geometry format:
# https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry
ut_stadium_wkt = "POINT(-97.7323317 30.2830764)"
# GeometryData is a protobuf container for GIS geometry information, the epsg in the spatial 
# reference defines the WGS-84 ellipsoid (`epsg=4326`) spatial reference (the latitude longitude 
# spatial reference most commonly used)
geometry_data = GeometryData(wkt=ut_stadium_wkt, proj=ProjectionData(epsg=4326))

# TimestampField is a query field that allows for making sql-like queries for information
# LTE is an enum that means less than or equal to the value in the query field
# Query data from August 25, 2019
time_filter = utils.pb_timestampfield(value=date(2019, 8, 25), rel_type=enum.FilterRelationship.LTE)

# the StacRequest is a protobuf message for making filter queries for data
# This search looks for any type of imagery hosted in the STAC service that intersects the austin 
# capital area of interest and was observed on or before August 25, 2019
stac_request = StacRequest(datetime=time_filter, intersects=geometry_data)

# search_one method requests only one item be returned that meets the query filters in the StacRequest 
# the item returned is a StacItem protobuf message. search_one, will only return the most recently 
# observed results that matches the time filter and spatial filter
stac_item = client.search_one(stac_request)

# get the thumbnail asset from the assets map. The other option would be a Geotiff, 
# with asset key 'GEOTIFF_RGB'
print("STAC id {}".format(stac_item.id))
asset = utils.get_asset(stac_item, asset_type=enum.AssetType.GEOTIFF)

# with save_dir as d:
d = os.getcwd()
filename = utils.download_asset(asset=asset, save_directory=d)
fp = filename
img = rasterio.open(fp)
show(img)
```
</details>
<br>

![Austin,TX](./img/austin.png)

## Concepts: Protobufs, gRPC, and Spatio Temporal Asset Catalogs (STAC)

The `nsl.stac` library connects to NearSpace Labs' Spatio Temporal Asset Catalog (STAC) which holds metadata about imagery. It uses the gPRC, a modern framwork for requesting services from the STAC server. The requests are formatted as a protocol buffer which is a binary structured format for serializing data.

Details about each are listed below:

**Definition of [STAC](https://stacspec.org/)**:
> The SpatioTemporal Asset Catalog (STAC) specification provides a common language to describe a range of geospatial information, so it can more easily be indexed and discovered.  A 'spatiotemporal asset' is any file that represents information about the earth captured in a certain space and time.

**Definition of [gRPC](https://grpc.io)**:
> gRPC is a modern open source high performance RPC framework that can run in any environment. It can efficiently connect services in and across data centers with pluggable support for load balancing, tracing, health checking and authentication. It is also applicable in last mile of distributed computing to connect devices, mobile applications and browsers to backend services.

**Definition of [Protocol Buffers (protobuf)](https://developers.google.com/protocol-buffers/)**:
> Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages.

To summarize:
- You can think of Protobuf as a strict data format like xml or JSON + linting, except Protobuf is a compact binary message with strongly typed fields
- gRPC is similar to REST + OpenAPI, except gRPC is an [RPC](https://en.wikipedia.org/wiki/Remote_procedure_call) framework that supports bi-directional streaming
- STAC is a  searchable catalog  of metadata for geospatial datasets or assets.

## Querying the STAC server

A STAC server contains metadata about and `asset` which is any type of data. The metadata can include a path, URL, or pointer to the data. The `nsl.stac` client searches for imagery or assets through metadata queries. To understand how and what to query, let's take a look at the metadata for a single item

```json
    id: "20190822T183518Z_746_POM1_ST2_P"
    collection: "NSL_SCENE"
    properties {
      type_url: "nearspacelabs.com/proto/st.protobuf.v1.NslDatast.protobuf.v1.NslData/st.protobuf.v1.NslData"
      value: "\n\340\014\n\03620190822T162258Z_TRAVIS_COUNTY\"\003 \352\0052\03520200702T102306Z_746_ST2_POM1:\03520190822T183518Z_746_POM1_ST2:\03520200702T101632Z_746_ST2_POM1:\03520200702T102302Z_746_ST2_POM1:\03520200702T102306Z_746_ST2_POM1B\03520190822T183518Z_746_POM1_ST2H\001R\374\n\n$\004\304{?\216\371\350=\376\377\306>\300\327\256\275\323rv?2\026*D3Qy6\177>\3675\000\000\200?\022\024\r+}\303\302\025\033;\362A\0353}\367\300%g\232\250@\022\024\r\026}\303\302\025\376?\362A\035\000\367\235@%\232\t\331?\022\024\r\351|\303\302\025\021A\362A\035M\370\033\301%g\016\226\277\022\024\r\201|\303\302\025\3709\362A\035\000\252\245@%\315\3547?\022\024\r\310|\303\302\025\245G\362A\035\232\315l\301%3\347\270\300\022\024\rq|\303\302\025\2149\362A\035\000\376o@%\000(\017@\022\024\rD|\303\302\025oD\362A\0353\323\302\301%\315\306\230\300\022\024\r\031|\303\302\025\035=\362A\035g\277$A%\000\340\231?\022\024\rE|\303\302\025\215I\362A\0353\275z\300%g\020\236\300\022\024\r\345{\303\302\0258C\362A\035\0008\242?%\232\231\226\277\022\024\r\010|\303\302\025!I\362A\0353\377\212\300%\000V\241\300\022\024\r|{\303\302\025\207F\362A\0353\203Y@%\315,\313\276\022\024\r\001{\303\302\025FJ\362A\035g^\025@%\315\010\214?\022\024\r\313z\303\302\025\353H\362A\0353\3377@%g\326\325\277\022\024\rjz\303\302\025\260@\362A\035\315F\006A%g\246[\277\022\024\r\035z\303\302\0254E\362A\035\232\001|@%\232!\265?\022\024\r\330y\303\302\025\320@\362A\0353Sa\300%\000@\245>\022\024\r\362y\303\302\025zE\362A\035\232\221\020\300%3U\206@\022\024\r\337y\303\302\025\210F\362A\035g\246l?%gf\234\276\022\024\r\335y\303\302\025aF\362A\035\000\260\023@%\315,#\277\022\024\r\321y\303\302\025\234F\362A\035\000 7@%\232!\221?\022\024\r\307y\303\302\025\177F\362A\035\232\371\371?%\315\224\225?\022\024\r\213y\303\302\025\350@\362A\0353\'\343\300%3g&\300\022\024\r\300y\303\302\025\tF\362A\035\315h\312@%g\266\013?\022\024\r_y\303\302\025\236A\362A\035\315\340\311@%3\363j>\022\024\r\271x\303\302\025G?\362A\0353\334\272\301%gb\201\300\022\024\r\307x\303\302\025WG\362A\035\000|6\301%\232\231i>\022\024\r\200x\303\302\025\016F\362A\035\315\007\244\301%\315L\000>\022\024\rqx\303\302\025jI\362A\035\315\254\007\301%\232E\247?\022\024\rjx\303\302\025(I\362A\035\232\305\000\301%\315L\'>\022\024\r\027x\303\302\025\356A\362A\035\232I\246?%\315\004\246\277\022\024\r\010x\303\302\025AB\362A\035\232y\305\300%\315\3740?\022\024\r\032x\303\302\0257D\362A\0353\003\275\277%\232\311.?\022\024\r\002x\303\302\025&C\362A\035\315\014\301\277%g*2@\022\024\r\361w\303\302\025\330B\362A\035\000T\347\300%\232\235\025\300\022\024\r\372v\303\302\025\030<\362A\0353\323\364?%gNt\300\022\024\r;w\303\302\025\273I\362A\03533\335>%\232\025\213?\022\024\r\324v\303\302\025QC\362A\035\315,\305\277%\232\375\035@\022\024\r\340v\303\302\025@G\362A\035\315@\234\300%\232)\342?\022\024\r\312v\303\302\025yC\362A\035\315\214\247\276%g\246\375>\022\024\r\222v\303\302\025\233A\362A\035\315\334\244?%g\366\035\277\022\024\r\256v\303\302\025\\F\362A\0353G\204@%\232A\017@\022\024\rov\303\302\025\215=\362A\035\232\325\340@%3\263\033\276\022\024\r\206v\303\302\025SC\362A\0353\263k?%3\363\177\276\022\024\r\267v\303\302\025NK\362A\035\315\0148\277%3\323\000>\022\024\r\255v\303\302\025kK\362A\035gf4\277%\000\312\201\277\022\024\r)v\303\302\025\316=\362A\035\232\271Z\277%\315\014\375\277\022\024\r_v\303\302\025\356H\362A\035\315\004n@%3\243\240\276\022\024\r7v\303\302\025\350H\362A\0353#\212@%g~\272?\022\024\r\314u\303\302\025Y;\362A\035\000\000F=%gF\253?\022\024\r\276u\303\302\025q>\362A\0353/\234\300%g\246T\277\022\024\r\266u\303\302\025\321>\362A\035\315 \272\300%3SW\300\022\024\r\307u\303\302\025\211A\362A\035\000$\264\300%3\243\r\277\022\024\r\360u\303\302\025RK\362A\0353\347\231@%\315\325\036\300\022\024\r\262u\303\302\025\035F\362A\0353\2633\276%\232i3?\032#m_3009743_sw_14_1_20160928_20161129\"Y\t&\2068NM\357\"A\021\003\3272rL\217IA\031\267G\014x\260\375\"A!\202I\225>\020\222IA*3\0221+proj=utm +zone=14 +datum=NAD83 +units=m +no_defs*\005\r\205[\"A2\005\r\000\356\\@:\005\r\227\210\306AB\005\r\205E\257@\022\315\001\n e502fe83507f0d28c826f33619a678e9\022\03120200806T033934Z_SWIFTERA\030\010 \377\377\377\377\377\377\377\377\377\001(A0\0018\340\025@\330\247\004H\270\275\004R\03620190822T162258Z_TRAVIS_COUNTYR\03120200701T112634Z_SWIFTERAR\03120200701T112634Z_SWIFTERAR\03120200701T112634Z_SWIFTERAX\263\027"
    }
    assets {
      key: "GEOTIFF_RGB"
      value {
        href: "https://api.nearspacelabs.net/download/20190822T162258Z_TRAVIS_COUNTY/Published/REGION_0/20190822T183518Z_746_POM1_ST2_P.tif"
        type: "image/vnd.stac.geotiff"
        eo_bands: RGB
        asset_type: GEOTIFF
        cloud_platform: GCP
        bucket_manager: "Near Space Labs"
        bucket_region: "us-central1"
        bucket: "swiftera-processed-data"
        object_path: "20190822T162258Z_TRAVIS_COUNTY/Published/REGION_0/20190822T183518Z_746_POM1_ST2_P.tif"
      }
    }
    assets {
      key: "THUMBNAIL_RGB"
      value {
        href: "https://api.nearspacelabs.net/download/20190822T162258Z_TRAVIS_COUNTY/Published/REGION_0/20190822T183518Z_746_POM1_ST2_P.png"
        type: "image/png"
        eo_bands: RGB
        asset_type: THUMBNAIL
        cloud_platform: GCP
        bucket_manager: "Near Space Labs"
        bucket_region: "us-central1"
        bucket: "swiftera-processed-data"
        object_path: "20190822T162258Z_TRAVIS_COUNTY/Published/REGION_0/20190822T183518Z_746_POM1_ST2_P.png"
      }
    }
    geometry {
      wkb: "\001\006\000\000\000\001\000\000\000\001\003\000\000\000\001\000\000\000\005\000\000\000\352\244L\267\311oX\300\316\340\320\247\234I>@\241\273\2606\267oX\300<\002\205\'EG>@\031\003\203\307\266nX\3001z\244\372\233G>@CCAI\306nX\300\326\013\351\023\343I>@\352\244L\267\311oX\300\316\340\320\247\234I>@"
      proj {
        epsg: 4326
      }
      envelope {
        xmin: -97.7466867683867
        ymin: 30.278398961994966
        xmax: -97.72990596574927
        ymax: 30.288621181865743
        proj {
          epsg: 4326
        }
      }
      simple: STRONG_SIMPLE
    }
    bbox {
      xmin: -97.7466867683867
      ymin: 30.278398961994966
      xmax: -97.72990596574927
      ymax: 30.288621181865743
      proj {
        epsg: 4326
      }
    }
    datetime {
      seconds: 1566498918
      nanos: 505476000
    }
    observed {
      seconds: 1566498918
      nanos: 505476000
    }
    created {
      seconds: 1596743811
      nanos: 247169000
    }
    updated {
      seconds: 1612193286
      nanos: 12850810
    }
    platform_enum: SWIFT_2
    platform: "SWIFT_2"
    instrument_enum: POM_1
    instrument: "POM_1"
    constellation: "UNKNOWN_CONSTELLATION"
    mission_enum: SWIFT
    mission: "SWIFT"
    gsd {
      value: 0.20000000298023224
    }
    eo {
    }
    view {
      off_nadir {
        value: 9.42326831817627
      }
      azimuth {
        value: -74.85270690917969
      }
      sun_azimuth {
        value: 181.26959228515625
      }
      sun_elevation {
        value: 71.41288757324219
      }
    }
```

