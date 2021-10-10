# Downloading

To download an asset use the `bucket` + `object_path` or the `href` fields from the asset, and download the data using the library of your choice. 

```test
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
```

``` bash
$ wget "https://api.nearspacelabs.net/download/20190822T162258Z_TRAVIS_COUNTY/Published/REGION_0/20190822T183518Z_746_POM1_ST2_P.tif"
```

There is also a download utility in the `nsl.stac.utils` module. Downloading from Google Cloud Storage buckets requires having defined your `GOOGLE_APPLICATION_CREDENTIALS` [environment variable](https://cloud.google.com/docs/authentication/getting-started#setting_the_environment_variable). Downloading from AWS/S3 requires having your configuration file or environment variables defined as you would for [boto3](https://boto3.amazonaws.com/v1/documentation/api/1.9.42/guide/quickstart.html#configuration). 

## Thumbnails
To downlad thumbnail assets follow the example below:

```python
import tempfile
from IPython.display import Image, display

from nsl.stac.client import NSLClient
from nsl.stac import utils, enum, StacRequest, GeometryData, ProjectionData

mlk_blvd_wkt = 'LINESTRING(-97.72842049283962 30.278624772098176,-97.72142529172878 30.2796624743974)'
geometry_data = GeometryData(wkt=mlk_blvd_wkt, 
                             proj=ProjectionData(epsg=4326))
time_filter = utils.pb_timestampfield(value=date(2019, 8, 25), rel_type=enum.FilterRelationship.LTE)
stac_request = StacRequest(intersects=geometry_data,
                           datetime=time_filter,
                           limit=3)

# get a client interface to the gRPC channel
client = NSLClient()

for stac_item in client.search(stac_request):
    # get the thumbnail asset from the assets map
    asset = utils.get_asset(stac_item, asset_type=enum.AssetType.THUMBNAIL)
    # (side-note delete=False in NamedTemporaryFile is only required for windows.)
    with tempfile.NamedTemporaryFile(suffix=".png", delete=False) as file_obj:
        utils.download_asset(asset=asset, file_obj=file_obj)
        display(Image(filename=file_obj.name))
```

![png](./img/README_18_0.png)
    
![png](./img/README_18_1.png)
    
![png](./img/README_18_2.png)
    
## Geotiffs

To download geotiff assets follow the example below:

```python
import os
import tempfile
from datetime import date
from nsl.stac import StacRequest, GeometryData, ProjectionData, enum
from nsl.stac import utils
from nsl.stac.client import NSLClient

client = NSLClient()

ut_stadium_wkt = "POINT(-97.7323317 30.2830764)"
geometry_data = GeometryData(wkt=ut_stadium_wkt, proj=ProjectionData(epsg=4326))

# Query data from before September 1, 2019
time_filter = utils.pb_timestampfield(value=date(2019, 9, 1), rel_type=enum.FilterRelationship.LTE)

stac_request = StacRequest(datetime=time_filter, intersects=geometry_data)

stac_item = client.search_one(stac_request)

# get the Geotiff asset from the assets map
asset = utils.get_asset(stac_item, asset_type=enum.AssetType.GEOTIFF)

with tempfile.TemporaryDirectory() as d:
    file_path = utils.download_asset(asset=asset, save_directory=d)
    print("{0} has {1} bytes".format(os.path.basename(file_path), os.path.getsize(file_path)))
```


```text
    20190826T190001Z_761_POM1_ST2_P.tif has 131373291 bytes
```

## Handling ddeadlines
The `search` method is a gRPC streaming request. It sends a single request to the server and then maintains an open connection to the server, which then pushes results to the client. This means that if you have a long running sub-routine that executes between each iterated result from `search` you may exceed the 15 second timeout. If you have a stac request so large that the results create a memory problem or the blocking behavior limits your application performance, then you will want to use `offset` and `limit` as described in [AdvancedExamples.md](./AdvancedExamples.md#limits-and-offsets).

Otherwise, an easy way to iterate through results without timing-out on long running sub-routines is to capture the `search` results in a `list`.

For example:





<details><summary>Expand Python Code Sample</summary>


```python
import os
import tempfile
from datetime import date
from nsl.stac import StacRequest, GeometryData, ProjectionData, enum
from nsl.stac.utils import download_asset, get_asset, pb_timestampfield
from nsl.stac.client import NSLClient


ut_stadium_wkt = "POINT(-97.7323317 30.2830764)"
geometry_data = GeometryData(wkt=ut_stadium_wkt, proj=ProjectionData(epsg=4326))

# Query data from before September 1, 2019
time_filter = pb_timestampfield(value=date(2019, 9, 1), rel_type=enum.FilterRelationship.LTE)

# limit is set to 2 here, but it would work if you set it to 100 or 1000
stac_request = StacRequest(datetime=time_filter, intersects=geometry_data, limit=2)

# get a client interface to the gRPC channel. This client singleton is threadsafe
client = NSLClient()

# collect all stac items in a list
stac_items = list(client.search(stac_request))

with tempfile.TemporaryDirectory() as d:
    for stac_item in stac_items:
        print("STAC item id: {}".format(stac_item.id))
        asset = get_asset(stac_item, asset_type=enum.AssetType.GEOTIFF)
        filename = download_asset(asset=asset, save_directory=d)
        print("saved {}".format(os.path.basename(filename)))
```


</details>




<details><summary>Expand Python Print-out</summary>


```text
    STAC item id: 20190826T190001Z_761_POM1_ST2_P
    saved 20190826T190001Z_761_POM1_ST2_P.tif
    STAC item id: 20190826T185933Z_747_POM1_ST2_P
    saved 20190826T185933Z_747_POM1_ST2_P.tif
```


</details>



## Differences between gRPC+Protobuf STAC and OpenAPI+JSON STAC
If you are already familiar with STAC, you'll need to know that gRPC + Protobuf STAC is slightly different from the JSON definitions. 

JSON is naturally a flexible format and with linters you can force it to adhere to rules. Protobuf is a strict data format and that required a few differences between the JSON STAC specification and the protobuf specification:

### JSON STAC Compared with Protobuf STAC

#### STAC Item Comparison
For Comparison, here is the [JSON STAC item field summary](https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#item-fields) and the [Protobuf STAC item field summary](https://geo-grpc.github.io/api/#epl.protobuf.v1.StacItem). Below is a table comparing the two:


| Field Name | STAC Protobuf Type                                                                                                       | STAC JSON Type                                                             |
| ---------- | ------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------- |
| id         | [string](https://geo-grpc.github.io/api/#string)                                                                         | string                                                                     |
| type       | NA                                                                                                                       | string                                                                     |
| geometry   | [GeometryData](https://geo-grpc.github.io/api/#epl.protobuf.v1.GeometryData)                                             | [GeoJSON Geometry Object](https://tools.ietf.org/html/rfc7946#section-3.1) |
| bbox       | [EnvelopeData](https://geo-grpc.github.io/api/#epl.protobuf.v1.EnvelopeData)                                             | [number]                                                                   |
| properties | [google.protobuf.Any](https://developers.google.com/protocol-buffers/docs/proto3#any)                                    | Properties Object                                                          |
| links      | NA                                                                                                                       | [Link Object]                                                              |
| assets     | [StacItem.AssetsEntry](https://geo-grpc.github.io/api/#epl.protobuf.v1.StacItem.AssetsEntry)                             | Map                                                                        |
| collection | [string](https://geo-grpc.github.io/api/#string)                                                                         | string                                                                     |
| title      | [string](https://geo-grpc.github.io/api/#string)                                                                         | Inside Properties                                                          |
| datetime   | [google.protobuf.Timestamp](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/timestamp.proto) | Inside Properties                                                          |
| observed   | [google.protobuf.Timestamp](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/timestamp.proto) | Inside Properties                                                          |
| processed  | [google.protobuf.Timestamp](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/timestamp.proto) | Inside Properties                                                          |
| updated    | [google.protobuf.Timestamp](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/timestamp.proto) | Inside Properties                                                          |
| duration   | [google.protobuf.Duration](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/duration.proto)   | Inside Properties                                                          |
| eo         | [Eo](https://geo-grpc.github.io/api/#epl.protobuf.v1.Eo)                                                                 | Inside Properties                                                          |
| sar        | [Sar](https://geo-grpc.github.io/api/#epl.protobuf.v1.Sar)                                                               | Inside Properties                                                          |
| landsat    | [Landsat](https://geo-grpc.github.io/api/#epl.protobuf.v1.Landsat)                                                       | Inside Properties                                                          |



#### Eo Comparison
For Comparison, here is the [JSON STAC Electro Optical field summary](https://github.com/radiantearth/stac-spec/tree/master/extensions/eo#item-fields) and the [Protobuf STAC Electro Optical field summary](https://geo-grpc.github.io/api/#epl.protobuf.v1.Eo). Below is a table comparing the two:

| JSON Field Name | JSON Data Type                                                                                 | Protobuf Field Name | Protobuf Data Type                                                                                                                |
| --------------- | ---------------------------------------------------------------------------------------------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| eo:bands        | [Band Object](https://github.com/radiantearth/stac-spec/tree/master/extensions/eo#band-object) | bands               | [Eo.Band](https://geo-grpc.github.io/api/#epl.protobuf.v1.Eo.Band)                                                                |
| eo:cloud_cover  | number                                                                                         | cloud_cover         | [google.protobuf.wrappers.FloatValue](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/wrappers.proto) |



### Updating the samples in this README
Use this README.ipynb notebook to update the README.md. Do not directly edit the README.md. It will be overwritten by output from `ipynb2md.py`. `ipynb2md.py` can be downloaded from this [gist](https://gist.github.com/davidraleigh/a24f637ccb018610a87aaacb12281452).

```bash
curl -o ipynb2md.py https://gist.githubusercontent.com/davidraleigh/a24f637ccb018610a87aaacb12281452/raw/009a50f29b920c7b00dcd4142c51bf6bf3c0cb3b/ipynb2md.py
```

Make your edits to the README.ipynb, in *Kernel->Restart & Run All* to confirm your changes worked, Save and Checkpoint, then run the python script `python ipynb2md.py -i README.ipynb`.


