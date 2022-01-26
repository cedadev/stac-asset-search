|   |   |
|---|---|
| **OpenAPI Specification** | [openapi.yaml](openapi.yaml) |
| **Conformance Class** | https://api.stacspec.org/v1.0.0-beta.2/asset-search#asset-search |
| **[Maturity Classification](https://github.com/radiantearth/stac-api-spec/blob/master/extensions.md#extension-maturity)** | Pilot |
  
A search endpoint provides the ability to query STAC Asset objects across collections.
It retrieves a group of Asset objects that match the provided parameters, wrapped in an  AssetCollection (which is a 
valid [GeoJSON FeatureCollection](https://tools.ietf.org/html/rfc7946#section-3.3) that contains STAC Asset objects). Several core
query parameters are defined by [OGC API - Features](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html), with
a few additions specified in this document.

## HTTP GET

### HTTP GET Example

Request:
```http
HTTP GET /search?bbox=-110,39.5,-105,40.5
```

Response with `200 OK`:
```json
{
    "type": "FeatureCollection",
    "features": [],
    "links": [
        {
            "rel": "next",
            "href": "http://api.cool-sat.com/search?page=2"
        }
    ]
}
```

## HTTP POST Example

Request:
```json
{
    "bbox": [-110, 39.5, -105, 40.5],
    "limit": 10
}
```

Response with `200 OK`:
```json
{
    "type": "FeatureCollection",
    "features": [],
    "links": [
        {
            "rel": "next",
            "href": "http://api.cool-sat.com/search?page=2"
        }
    ]
}
```
