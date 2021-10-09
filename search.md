# Search

## Simple query by id

The simplest query is a `StacRequest` constructor without parameters. If we know the `id` of a STAC item, we can construct the `StacRequest` as follows:

```python
from nsl.stac.client import NSLClient
from nsl.stac import StacRequest
# get a client interface to the gRPC channel
client = NSLClient()

stac_request = StacRequest(id='20190822T183518Z_746_POM1_ST2_P')

# for this request we might as well use the search one, as STAC ids ought to be unique
stac_item = client.search_one(stac_request)
print(stac_item)
```

The request returns a lengthy record of the stac item . Although `stac_item` is a protobuf object, it has a `__str__` method that returns a JSON-like object. The example below contains the following:

- [GeometryData](https://geo-grpc.github.io/api/#epl.protobuf.v1.GeometryData) which is defined with a WGS-84 well-known binary geometry
- [EnvelopeData](https://geo-grpc.github.io/api/#epl.protobuf.v1.EnvelopeData) which is also WGS-84
- [Timestamp](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/timestamp.proto) Google's protobuf unix time format
- [Eo](https://geo-grpc.github.io/api/#epl.protobuf.v1.Eo) for electro-optical sensor details
- [Landsat](https://geo-grpc.github.io/api/#epl.protobuf.v1.Landsat) for Landsat specific details
- an array map of [StacItem.AssetsEntry](https://geo-grpc.github.io/api/#epl.protobuf.v1.StacItem.AssetsEntry) with each [Asset](https://geo-grpc.github.io/api/#epl.protobuf.v1.Asset) containing details about [AssetType](https://geo-grpc.github.io/api/#epl.protobuf.v1.AssetType), Electro Optical [Band enums](https://geo-grpc.github.io/api/#epl.protobuf.v1.Eo.Band) (if applicable), and other details for downloading and interpreting data

You may have noticed that the [Asset](https://geo-grpc.github.io/api/#epl.protobuf.v1.Asset) has a number of additional parameters not included in the JSON STAC specification. 


<details><summary>View STAC item record</summary>

```text
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
</details>

## Spatial Queries

### Query by bounding box

The STAC specification has a bounding box `bbox` specification for STAC items. We can use a bouding box to make a STAC request. We define an [EnvelopeData](https://geo-grpc.github.io/api/#epl.protobuf.v1.EnvelopeData) protobuf object. The advantage of doing this is that we specify other projections besides WGS84, which you cannot do with a JSON STAC request.

```python
from nsl.stac import StacRequest, EnvelopeData, ProjectionData
from nsl.stac.client import NSLClient

client = NSLClient()

# define our area of interest bounds using the xmin, ymin, xmax, ymax coordinates of an area on 
# the WGS-84 ellipsoid
neighborhood_box = (-97.7352547645, 30.27526474757116, -97.7195692, 30.28532)
# here we define our envelope_data protobuf with bounds and a WGS-84 (`epsg=4326`) spatial reference
envelope_data = EnvelopeData(xmin=neighborhood_box[0], 
                             ymin=neighborhood_box[1], 
                             xmax=neighborhood_box[2], 
                             ymax=neighborhood_box[3],
                             proj=ProjectionData(epsg=4326))
# Search for data that intersects the bounding box
stac_request = StacRequest(bbox=envelope_data)


for stac_item in client.search(stac_request):
    print("STAC item id: {}".format(stac_item.id))
```

The query returns the STAC ids of 10 items.

```text
    STAC item id: 20200703T174443Z_650_POM1_ST2_P
    STAC item id: 20200703T174303Z_595_POM1_ST2_P
    STAC item id: 20200703T174258Z_592_POM1_ST2_P
    STAC item id: 20200703T174234Z_579_POM1_ST2_P
    STAC item id: 20200703T174155Z_558_POM1_ST2_P
    STAC item id: 20200703T174034Z_516_POM1_ST2_P
    STAC item id: 20200703T174032Z_515_POM1_ST2_P
    STAC item id: 20200703T174028Z_513_POM1_ST2_P
    STAC item id: 20200703T174021Z_509_POM1_ST2_P
    STAC item id: 20200703T174019Z_508_POM1_ST2_P
```

### Query By Geomtery

#### Query with geojson

We can search by geometry instead of bounding box. We'll use geojson to define our area of interest as a [GeometryData](https://geo-grpc.github.io/api/#epl.protobuf.v1.GeometryData) protobuf. GeometryData can be defined using geojson, wkt, wkb, or as an ESRI shape:

```python
import json
import requests

from nsl.stac import StacRequest, GeometryData, ProjectionData
from nsl.stac.client import NSLClient
client = NSLClient()

# request the geojson footprint of Travis County, Texas
url = "http://raw.githubusercontent.com/johan/world.geo.json/master/countries/USA/TX/Travis.geo.json"
r = requests.get(url)
travis_geojson = json.dumps(r.json()['features'][0]['geometry'])
# create our GeometryData protobuf from geojson string and WGS-84 ProjectionData protobuf
geometry_data = GeometryData(geojson=travis_geojson, 
                             proj=ProjectionData(epsg=4326))
# Search for data that intersects the geojson geometry and limit results 
# to 2 (instead of default of 10)
stac_request = StacRequest(intersects=geometry_data, limit=2)
# collect the ids from STAC items to compare against results from wkt GeometryData
geojson_ids = []

# get a client interface to the gRPC channel
client = NSLClient()
for stac_item in client.search(stac_request):
    print("STAC item id: {}".format(stac_item.id))
    geojson_ids.append(stac_item.id)
```

The query returns two items:

```text
    STAC item id: 20201001T211834Z_2012_POM1_ST2_P
    STAC item id: 20201001T211832Z_2011_POM1_ST2_P
```

#### Query with [Well Known Text (WKT)](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry)

Using the same geometry as the example above, but with WKT geometry instead of a geojson:

```python
# Same geometry as above, but a wkt geometry instead of a geojson
travis_wkt = "POLYGON((-97.9736 30.6251, -97.9188 30.6032, -97.9243 30.5703, -97.8695 30.5484, \
              -97.8476 30.4717, -97.7764 30.4279, -97.5793 30.4991, -97.3711 30.4170, \
              -97.4916 30.2089, -97.6505 30.0719, -97.6669 30.0665, -97.7107 30.0226, \
              -98.1708 30.3567, -98.1270 30.4279, -98.0503 30.6251))" 
geometry_data = GeometryData(wkt=travis_wkt, 
                             proj=ProjectionData(epsg=4326))
stac_request = StacRequest(intersects=geometry_data, limit=2)
for stac_item in client.search(stac_request):
    print("STAC item id: {0} from wkt filter intersects result from geojson filter: {1}"
          .format(stac_item.id, stac_item.id in geojson_ids))
```

The query returns two items:

```text
    STAC item id: 20201001T211834Z_2012_POM1_ST2_P from wkt filter intersects result from geojson filter: True
    STAC item id: 20201001T211832Z_2011_POM1_ST2_P from wkt filter intersects result from geojson filter: True
```

## Temporal Queries

Time query filters use `>`, `>=`, `<`, `<=`, `==`, `!=` operations and inclusive and exclusive range requests. Queries use a [TimestampFilter](https://geo-grpc.github.io/api/#epl.protobuf.v1.TimestampFilter), we define the value using the `value` field or the `start`&`end` fields.

We define a relationship type using the `rel_type` field and the [FilterRelationship](https://geo-grpc.github.io/api/#epl.protobuf.v1.FilterRelationship) enum values of `EQ`, `LTE`, `GTE`, `LT`, `GT`, `BETWEEN`, `NOT_BETWEEN`, or `NEQ`.

Note that `nsl.stac` uses Google's [Timestamp proto](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/timestamp.proto) to define the temporal aspect of STAC items, where time is stored with an `int64` for seconds and an `int32` for nanoseconds relative to an epoch at UTC midnight on January 1, 1970.

The time fields on a [StacItem](https://geo-grpc.github.io/api/#epl.protobuf.v1.StacItem), you'll notice that `datetime`, `observed`, `created`, and `processed` all use the Timestamp Protobuf object.


### Query everything after a date

```python
from datetime import date, datetime, timezone
from nsl.stac.client import NSLClient
from nsl.stac import utils, StacRequest, enum

# make a filter that selects all data on or after August 21st, 2019
value = date(2019, 8, 21)
time_filter = utils.pb_timestampfield(value=value, rel_type=enum.FilterRelationship.GTE)
stac_request = StacRequest(datetime=time_filter, limit=2)

# get a client interface to the gRPC channel
client = NSLClient()
for stac_item in client.search(stac_request):
    print("STAC item date, {0}, is after {1}: {2}".format(
        datetime.fromtimestamp(stac_item.observed.seconds, tz=timezone.utc).isoformat(),
        datetime.fromtimestamp(time_filter.value.seconds, tz=timezone.utc).isoformat(),
        stac_item.observed.seconds > time_filter.start.seconds))
```
The result shows the datetime of the STAC item, the datetime of the query and a confirmation that the items satisfy the query filter. Notice the warning, this is because our date doesn't have a timezone associated with it. By default we assume UTC.

```text
    STAC item date, 2021-02-07T20:29:00+00:00, is after 2019-08-21T00:00:00+00:00: True
    STAC item date, 2021-02-07T20:28:58+00:00, is after 2019-08-21T00:00:00+00:00: True
```

#### Query everything between two dates

This range request selects data between two dates using the `start` and `end` parameters instead of the `value` parameter:

```python
from datetime import datetime, timezone, timedelta
from nsl.stac.client import NSLClient
from nsl.stac import utils, enum, StacRequest
# Query data from August 1, 2019
start = datetime(2019, 8, 1, 0, 0, 0, tzinfo=timezone.utc)
# ... up until August 10, 2019
end = start + timedelta(days=9)
time_filter = utils.pb_timestampfield(start=start, end=end, rel_type=enum.FilterRelationship.BETWEEN)

stac_request = StacRequest(datetime=time_filter, limit=2)

# get a client interface to the gRPC channel
client = NSLClient()
for stac_item in client.search(stac_request):
    print("STAC item date, {0}, is before {1}: {2}".format(
        datetime.fromtimestamp(stac_item.observed.seconds, tz=timezone.utc).isoformat(),
        datetime.fromtimestamp(time_filter.end.seconds, tz=timezone.utc).isoformat(),
        stac_item.observed.seconds < time_filter.end.seconds))
```
The request returns STAC items between the dates of Aug 1 2019 and Aug 10 2019. Also, notice there's no warnings as we defined our utc timezone on the datetime objects.

```text
    STAC item date, 2019-08-06T20:42:53+00:00, is before 2019-08-10T00:00:00+00:00: True
    STAC item date, 2019-08-06T20:42:51+00:00, is before 2019-08-10T00:00:00+00:00: True
```

#### Select data for one day

We can search on a specific day using Python's `datetime.date` for the `value` and `rel_type` set to  use equals (`FilterRelationship.EQ`). 

Python's `datetime.datetime` is a specific value and when combined with `EQ`, the query requires that the time relationship match down to the second. However, `datetime.date` is specific down to the day, the filter is created for the entire day. This checks request everything from the start until the end of the 8th of August, in the Austin, Texas' timezone (UTC -6).

```python
from datetime import datetime, timezone, timedelta, date
from nsl.stac.client import NSLClient
from nsl.stac import utils, enum, StacRequest
# Query all data for the entire day of August 6, 2019
value = date(2019, 8, 6)
# if you omit this tzinfo from the pb_timestampfield function, the default for tzinfo 
# is assumed to be utc 
texas_utc_offset = timezone(timedelta(hours=-6))
time_filter = utils.pb_timestampfield(rel_type=enum.FilterRelationship.EQ,
                                      value=value,
                                      tzinfo=texas_utc_offset)

stac_request = StacRequest(datetime=time_filter, limit=2)

# get a client interface to the gRPC channel
client = NSLClient()
for stac_item in client.search(stac_request):
    print("STAC item date, {0}, is before {1}: {2}".format(
        datetime.fromtimestamp(stac_item.observed.seconds, tz=timezone.utc).isoformat(),
        datetime.fromtimestamp(time_filter.end.seconds, tz=texas_utc_offset).isoformat(),
        stac_item.observed.seconds < time_filter.end.seconds))
    print("STAC item date, {0}, is after {1}: {2}".format(
        datetime.fromtimestamp(stac_item.observed.seconds, tz=timezone.utc).isoformat(),
        datetime.fromtimestamp(time_filter.start.seconds, tz=texas_utc_offset).isoformat(),
        stac_item.observed.seconds > time_filter.start.seconds))
```

The query returns results between `2019-08-06T00:00:00-06:00` and `2019-08-06T23:59:59-06:00`.

```text
    STAC item date, 2019-08-06T20:42:53+00:00, is before 2019-08-06T23:59:59-06:00: True
    STAC item date, 2019-08-06T20:42:53+00:00, is after 2019-08-06T00:00:00-06:00: True
    STAC item date, 2019-08-06T20:42:51+00:00, is before 2019-08-06T23:59:59-06:00: True
    STAC item date, 2019-08-06T20:42:51+00:00, is after 2019-08-06T00:00:00-06:00: True
```

[**< Getting Started**](./README.md)    [**Download >**](./download.md)
