# NSGI coordinate reference system transformation examples

The [NSGI](https://www.nsgi.nl/) is responsible for the CRSs (coordinate reference systems) of the Netherlands and their relations with international CRSs. The NSGI also gives advice and guidance about the usage of these CRSs. The Guidance document (in: Dutch) ["Handreiking Gebruik coordinaatreferentiesystemen bij uitwisseling en visualisatie van geo-informatie"](https://docs.geostandaarden.nl/crs/crs/) and the [EPSG repository](https://epsg.org/home.html) has documented the advices and guidelines that are relevant for exchange and visualisation of geoinformation.

## Goal

The goal of this GitHub organisation and the repositories associated with it is to give a technical perspective into how to implement the advice and guidelines.

## Relations

The image below shows the relevant relations between the Dutch and international CRSs. The transformation path between these CRSs can be ambiguous and dependent on the use case, especially for transformations between dynamic international CRSs and plate-fixed CRSs. NSGI has recommendations for the transformation procedures for use within the Netherlands. Defining these relations by making specific transformation rules within `proj.db` enables users to transform coordinates within the bounds of the European Netherlands and Caribbean Netherlands, including the Exclusive Economic Zones in a more accurate and consistent way.

![relations](https://raw.githubusercontent.com/GeodetischeInfrastructuur/transformations/main/supported-transformations-nsgi.drawio.svg)

## Products

The products that are available, within the context of CRSs and their relations, are layered on top of each other. At the core of it sits the modified [`proj.db`](https://proj.org), at the edge of it sits the "final product"; the deployed [coordinate transformation API](https://api.transformation.nsgi.nl/v2/).

Working with CRSs and implementing these can be tricky. By showing this layering of different technical solutions, in the image below, we hope to provide a clear breakdown of the relations between the products that are available. It is also important to note that these products can be used on their own, and that this layering can be used as an example of how we used our products for deploying the [coordinate transformation API](https://api.transformation.nsgi.nl/v2/).

![products](products.drawio.svg)

### :earth_africa: proj.db

The `proj.db` is a technical representation of the EPSG dataset. The `proj.db` is used by PROJ to implement these defined operations, making it possible to transform coordinates between CRSs. This makes the`proj.db` the first technical component available that can be modified. Other applications also leverage PROJ (and thus `proj.db`) for CRS handling and transformations (i.e. [QGIS](https://qgis.org/), [Mapserver](https://mapserver.org/), etc.). So if you want to change the CRS handling and transformations of these applications, then updating the `proj.db` would be the preferred technical solution.

> **example**
>
> ```bash
> curl -L -H "Accept: application/octet-stream" https://github.com/GeodetischeInfrastructuur/transformations/releases/download/1.0.0/proj.db -o proj.db
> ```

The modified `proj.db` is available to download through _github.com_ as:

1. [Dockerfile](https://github.com/GeodetischeInfrastructuur/transformations/blob/main/Dockerfile)
1. [Docker image](https://github.com/GeodetischeInfrastructuur/transformations/pkgs/container/transformations)
1. [Release download](https://github.com/GeodetischeInfrastructuur/transformations/releases)

### :mag: Geodense

A straight line in reality (geodesic) is usually not a straight line in a projected CRS. The deviation in the CRS dependents on the location, orientation and length of line segments. For unprojected CRS, the deviation depends on the projection used for visualisation or the computation method that is used. The deviation can cause inconsistency between the same data in different CRSs or compared to reality. To avoid topology issues, line segments can be densified by introducing additional vertices. 

Geodense is used to check the density and to densify `LINESTRING` and `POLYGON` GeoJSON geometries. In other words, this tool can be used to check the magnitude of the deviation of a line segment and introduce new points to that define the correct trajectory. It pre-processes certain geometry types so transformation doesn't affect the topology.

> :exclamation: For context:
> * a straight line in RD (`EPSG:28992`) with a length of 2 km could have a deviation up to 5 mm in reality (geodesic). For a length of 20 km the deviation can be up to 26 cm. [[1]](https://gnss-data.kadaster.nl/misc/docs/langelijnenadvies.pdf)
> * a straight line in Webmercator (`EPSG:3857`) with a length of 2 km could have a deviation up to 9,7 cm in reality (geodesic). For a length of 20 km the deviation can be up to 9,7 m. [[1]](https://gnss-data.kadaster.nl/misc/docs/langelijnenadvies.pdf)

For an example how to use Geodense one can look at [coordinate-transformation-api](https://github.com/GeodetischeInfrastructuur/coordinate-transformation-api).

> **example**
>
> ```bash
> pip install geodense
> ```
>
> ```python
> from geodense.lib import check_density_geometry_coordinates
> from geodense.models import DenseConfig
> from geodense.types import Nested, ReportLineString
> from geojson_pydantic import Feature
> from pyproj import CRS
>
> feature: Feature = {"type": "Feature","properties": {},"geometry": {"type": "LineString","coordinates": [[156264.9064,601302.5889],[165681.9644,605544.3132]]}}
> c = DenseConfig(CRS.from_epsg(28992))
> result: Nested[ReportLineString] = check_density_geometry_coordinates(feature['geometry']['coordinates'], c)
> print(result)
> ```

Geodense is available as:

1. [Code](https://github.com/GeodetischeInfrastructuur/geodense)
1. [PyPI package](https://pypi.org/project/geodense/0.0.1a9/)

### :computer: Coordinate Transformation API

> :warning: The Coordinate Transformation API makes use of pyproj. Pyproj has it's own PROJ 'build in' that needs to be updated. By default this can be found `/usr/local/lib/python3.11/site-packages/pyproj/proj_dir/share/proj/proj.db`.

The [coordinate-transformation-api](https://github.com/GeodetischeInfrastructuur/coordinate-transformation-api) is written in Python where the modified `proj.db` and Geodense are used together with specific code to create an API that will transform certain CRS and is focused on the European and Caribbean Netherlands including the Exclusive Economic Zones.

> :warning: The Coordinate Transformation API offers transformations that are not possible when using only PROJ.

The code for the Coordinate Transformation API is available at:

1. [github.com](https://github.com/GeodetischeInfrastructuur/coordinate-transformation-api)
1. [Dockerfile](https://github.com/GeodetischeInfrastructuur/coordinate-transformation-api/blob/main/Dockerfile)

### :whale2: Docker

A Docker image to run your own Coordinate Transformation API, as a backend service, is also available at _github.com_. Running the Coordinate Transformation API in your own container environment gives you full control over the availability of the API.

* [Docker image](https://github.com/GeodetischeInfrastructuur/coordinate-transformation-api/pkgs/container/coordinate-transformation-api)

### :globe_with_meridians: Coordinate Transformation API

We also host this Coordinate Transformation API on the following URL: [`https://api.transformation.nsgi.nl/v2/`](https://api.transformation.nsgi.nl/v2/). This can be used for datasets of limited size.

> **example**
>
> :exclamation: The API should not be used for large amouts of data or when high availability is required. The API could go down for maintenance, will throttle under 'high' load, and so on. 
>
> :heavy_check_mark: For implementing these CRSs and transformations in a test environment either:
>
> 1. spin up a container with the Coordinate Transformation API
> 1. use the modified `proj.db` in the PROJ environment

1. [OpenAPI Specification](https://api.transformation.nsgi.nl/v2/openapi?f=html)

## API Conformance

The Coordinate Transformation API conforms to certain degree to the following specifications:

| spec | compliance |
| --- | --- |
| OGC-API-Common | [report](https://github.com/GeodetischeInfrastructuur/coordinate-transformation-api/blob/main/docs/OGC-API-Common.md) |
| OGC-API-Features CRS | [report](https://github.com/GeodetischeInfrastructuur/coordinate-transformation-api/blob/main/docs/OGC-API-Features-CRS.md) |
| NL-API | [report](https://github.com/GeodetischeInfrastructuur/coordinate-transformation-api/blob/main/docs/NL-API.md) |
| KP-API geospatial | [report](https://github.com/GeodetischeInfrastructuur/coordinate-transformation-api/blob/main/docs/KP-API-geospatial.md) |

> :warning: The coordinate transformation API only performs coordinate transformation of the user input and doesn't contain something like a 'state'. Therefore, it doesn't conform to a traditional data object or feature collection API (like OGC API features), on which the above specifications are primarily focussed. So specification requirements or recommendations focussed on certain type of query parameters cannot be applied. The reason to include these reports regarding compliance (at least our current assumption) is that this can/will be used in environments that implement those kind of APIs. Having these reports will highlight the differences and similarities between the APIs

## Examples

### QGIS

The following instructions are for configuring QGIS on Windows to use the modified `proj.db` by NSGI.

Steps:

1. Obtain PROJ data directory path: run following Python code from the QGIS Python console and copy paste the output:

    ```python
    import pyproj;print(pyproj.datadir.get_data_dir())
    ```

1. Close QGIS and download the modified `proj.db` and correction grids with the following PowerShell or Bash script (run in console/terminal with elevated privileges):

    ```powershell
    # powershell script
    $PROJ_DIR = XXXX  # use the proj data directory path obtained at step 1
    cp $PROJ_DIR\proj.db $PROJ_DIR\proj.db.bak # backup original proj.db
    invoke-webrequest -uri https://cdn.proj.org/nl_nsgi_nlgeo2018.tif -outfile "$PROJ_DIR\nl_nsgi_nlgeo2018.tif"
    invoke-webrequest -uri https://cdn.proj.org/nl_nsgi_rdcorr2018.tif -outfile "$PROJ_DIR\nl_nsgi_rdcorr2018.tif"
    invoke-webrequest -uri https://cdn.proj.org/nl_nsgi_rdtrans2018.tif -outfile "$PROJ_DIR\nl_nsgi_rdtrans2018.tif"
    $asset=Invoke-Webrequest -uri https://api.github.com/repos/GeodetischeInfrastructuur/transformations/releases/latest | ConvertFrom-Json | select -Expand assets | where-object { $_.name -eq 'proj.db'}
    Invoke-Webrequest -uri $($asset.url) -Headers @{'Accept'='application/octet-stream';} -outfile "$PROJ_DIR\proj.db"
    ```

    ```bash
    # bash script
    PROJ_DATA_DIR=XXXX # use the proj data directory path obtained at step 1
    cp $PROJ_DATA_DIR/proj.db $PROJ_DATA_DIR/proj.db.bak # backup original proj.db
    curl -sL -o "${PROJ_DATA_DIR}/nl_nsgi_nlgeo2018.tif" https://cdn.proj.org/nl_nsgi_nlgeo2018.tif
    curl -sL -o "${PROJ_DATA_DIR}/nl_nsgi_rdcorr2018.tif" https://cdn.proj.org/nl_nsgi_rdcorr2018.tif
    curl -sL -o "${PROJ_DATA_DIR}/nl_nsgi_rdtrans2018.tif" https://cdn.proj.org/nl_nsgi_rdtrans2018.tif
    curl -sL -H "Accept: application/octet-stream" $(curl -s "https://api.github.com/repos/GeodetischeInfrastructuur/transformations/releases/latest" | jq -r '.assets[] | select(.name=="proj.db").url') -o "${PROJ_DATA_DIR}/proj.db"
    ```

1. Verify if QGIS is using the modified `proj.db`. Run the following Python script in the QGIS console, the output should read: _`proj db is configured correctly`_:

    ```py
    from pyproj import CRS, Transformer
    in_crs=CRS.from_epsg(7931)
    out_crs=CRS.from_epsg(28992)
    t=Transformer.from_crs(in_crs, out_crs, always_xy=True)
    input_point = (5, 52, 43)
    expected_output_point = (128410.0958, 445806.496, 43.0)
    output_point=tuple(map(lambda x: float("{:.4f}".format(x)),t.transform(*input_point)))
    assert output_point == expected_output_point, f"expected output is {expected_output_point}, was {output_point}"
    print("proj db is configured correctly")
    ```

> **NOTE:** the original `proj.db` file can be restored by running the following (in an elevated Powershell console):
>
> ```powershell
> $PROJ_DIR = XXXX  # use the proj data directory path obtained at step 1
> cp $PROJ_DIR\proj.db.bak$ PROJ_DIR\proj.db # restore backup
> ```
