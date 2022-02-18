# STAC API - Asset Search

- [STAC API - Asset Search](#stac-api---item-search)
  - [Link Relations](#link-relations)
  - [Endpoints](#endpoints)
  - [Query Parameters and Fields](#query-parameters-and-fields)
    - [Query Examples](#query-examples)
    - [Query Parameter Table](#query-parameter-table)
  - [Response](#response)
    - [Pagination](#pagination)
  - [HTTP Request Methods and Content Types](#http-request-methods-and-content-types)
    - [GET](#get)
    - [POST](#post)
      - [PUT / PATCH / DELETE](#put--patch--delete)
  - [Extensions](#extensions)
    - [Fields Extension](#fields-extension)
    - [Sort Extension](#sort-extension)
    - [Context Extension](#context-extension)
    - [Filter Extension](#filter-extension)
    - [Query Extension](#query-extension)

- **OpenAPI specification:** [openapi.yaml](openapi.yaml)
- **Conformance Class:** https://api.stacspec.org/v1.0.0-beta.2/asset-search#asset-search
- **[Maturity Classification](https://github.com/radiantearth/stac-api-spec/blob/master/extensions.md#extension-maturity):** Pilot
- **Dependencies**: [STAC API - Core](../core)
- **Examples**: [examples.md](examples.md)

A search endpoint provides the ability to query STAC Asset objects across collections.
It retrieves a group of Asset objects that match the provided parameters, wrapped in an  AssetCollection (which is a 
valid [GeoJSON FeatureCollection](https://tools.ietf.org/html/rfc7946#section-3.3) that contains STAC Asset objects). Several core
query parameters are defined by [OGC API - Features](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html), with
a few additions specified in this document.

The Asset Search endpoint follows Item Search intentionally defining only a limited group of operations. It is expected that 
most behavior will be defined in [Extensions](#extensions). See [Item Search](https://github.com/radiantearth/stac-api-spec/blob/master/item-search/README.md) 

Implementing `GET /search` is **required**, `POST /search` is optional, but recommended.

## Link Relations

This conformance class also requires implementation of the link relations in the [STAC API - Asset Search](../core) conformance class.

The following Link relations shall exist in the Landing Page (root).

| **rel**.       | **href**        | **From**                | **Description**                   |
| -------------- | --------------- | ----------------------- | --------------------------------- |
| `asset search` | `/asset/search` | STAC API - Asset Search | URI for the Asset Search endpoint |

The `asset search` link relation shall have a `type` of `application/geo+json` and a `method` of `GET`, and may also
a link with a `method` of `POST` if the server supports it.

## Endpoints

This conformance class also requires for the endpoints in the [STAC API - Core](../core) conformance class to be implemented.

| Endpoint        | Returns          | Description           |
| --------------- | ---------------- | --------------------- |
| `/asset/search` | Asset Collection | Asset Search endpoint |
 
## Query Parameters and Fields

The following list of parameters is used to narrow search queries. They can all be represented as query 
string parameters in a GET request, or as JSON entity fields in a POST request. For filters that represent 
a set of values, query parameters must use comma-separated string values with no enclosing brackets 
(\[ or \]) and no whitespace between values, and JSON entity attributes must use JSON Arrays. 

### Query Examples

```http
GET /asset/search?items=81420fb98d5c2bdd5814c5879543b300,fdfvcvfvf98d5c2bdafvfv9fdfvdv0&bbox=-10.415,36.066,3.779,44.213&limit=200&datetime=2017-05-05T00:00:00Z
```

```json
{
    "items": ["81420fb98d5c2bdd5814c5879543b300","fdfvcvfvf98d5c2bdafvfv9fdfvdv0"],
    "bbox": [10.415,36.066,3.779,44.213],
    "limit": 200,
    "datetime": "2017-05-05T00:00:00Z"
}
```

For more examples see [examples.md](examples.md).

### Query Parameter Table

The core parameters for STAC asset search are defined by OAFeat, and STAC adds a few parameters for convenience.

| Parameter   | Type             | Source API | Description                                                                                                                                                                     |
| ----------- | ---------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| limit       | integer          | OAFeat     | The maximum number of results to return (page size).                                                                                                                            |
| bbox        | \[number]        | OAFeat     | Requested bounding box.                                                                                                                                                         |
| datetime    | string           | OAFeat     | Single date+time, or a range ('/' separator), formatted to [RFC 3339, section 5.6](https://tools.ietf.org/html/rfc3339#section-5.6). Use double dots `..` for open date ranges. |
| intersects  | GeoJSON Geometry | STAC       | Searches items by performing intersection between their geometry and provided GeoJSON geometry.  All GeoJSON geometry types must be supported.                                  |
| ids         | \[string]        | STAC       | Array of Asset ids to return.                                                                                                                                                    |
| items       | \[string]        | STAC       | Array of one or more Item ids that each matching Asset must be in.                                                                                                         |

See [examples](examples.md) for some example requests.

**limit** The limit parameter follows the same semantics of the OAFeat Item resource limit parameter. The value is 
a suggestion to the server as to the maximum number of Item objects the
client would prefer in the response. The OpenAPI specification defines the default and maximum values
for this parameter. The base specifications define these with a default of 10 and a maximum of 10000, but implementers
may choose other values to advertise through their `service-desc` endpoint.  If the limit parameter value is greater
than the advertised maximum limit, the server shall return the maximum possible number of items (ideally, the number 
as the advertised maximum limit), rather than responding with an error.

Only one of either **intersects** or **bbox** may be specified.  If both are specified, a 400 Bad Request response 
must be returned. 

**datetime** The datetime parameter use the same allowed values as the 
[OAF datetime](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html#_parameter_datetime) parameter. 
This allows for either a single [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) datetime or an 
open or closed interval that also uses RFC 3339 datetimes. Additional details about this parameter can be 
found in the [Implementation Recommendations](../implementation.md#datetime-parameter-handling).

**bbox** Represented using either 2D or 3D geometries. The length of the array must be 2\*n where 
*n* is the number of dimensions. The array contains all axes of the southwesterly most extent 
followed by all axes of the northeasterly most extent specified in Longitude/Latitude or 
Longitude/Latitude/Elevation based on [WGS 84](http://www.opengis.net/def/crs/OGC/1.3/CRS84). 
When using 3D geometries, the elevation of the southwesterly most extent is the minimum elevation 
in meters and the elevation of the northeasterly most extent is the maximum. When filtering with 
a 3D bbox over Items with 2D geometries, it is assumed that the 2D geometries are at 
elevation 0.

## Response

The response to a request (GET or POST) to the search endpoint must always be an 
[AssetCollection]() object - a valid [GeoJSON 
FeatureCollection](https://tools.ietf.org/html/rfc7946#section-3.3) that consists entirely of STAC 
[Asset]() objects. 

### Pagination

OGC API supports paging through hypermedia links and STAC follows the same pattern for the cross collection search. For 
GET requests, a link with `rel` type `next` is supplied.  This link may contain any URL parameter that is necessary 
for the implementation to understand how to provide the next page of results, eg: `page`, `next`, `token`, etc.  The 
parameter name is defined by the implementor and is not necessarily part of the API specification.  For example:

```json
{
    "type": "FeatureCollection",
    "features": [],
    "links": [
        {
            "rel": "next",
            "href": "http://api.cool-sat.com/search?page=3"
        },
        {
            "rel": "prev",
            "href": "http://api.cool-sat.com/search?page=1"
        }
    ]
}
```

The href may contain any arbitrary URL parameter:

- `http://api.cool-sat.com/search?page=2`
- `http://api.cool-sat.com/search?next=8a35eba9c`
- `http://api.cool-sat.com/search?token=f32890a0bdb09ac3`

Implementations may also add link relations `prev`, `first`, and `last`, though these are not required and may
be infeasible to implement in some data stores.

OAFeat does not support POST requests for searches, however the STAC API spec does. Hypermedia links are not designed 
for anything other than GET requests, so providing a next link for a POST search request becomes problematic. STAC has 
decided to extend the Link object to support additional fields that provide hints to the client as to how it must 
execute a subsequent request for the next page of results.

The following fields have been added to the Link object specification for the API spec:

| Parameter | Type    | Description                                                                                                                                                  |
| --------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| method    | string  | The HTTP method of the request, usually `GET` or `POST`. Defaults to `GET`                                                                                   |
| headers   | object  | A dictionary of header values that must be included in the next request                                                                                      |
| body      | object  | A JSON object containing fields/values that must be included in the body of the next request                                                                 |
| merge     | boolean | If `true`, the headers/body fields in the `next` link must be merged into the original request and be sent combined in the next request. Defaults to `false` |

The implementor has the freedom to decide exactly how to apply these extended fields for their particular pagination 
mechanism.  The same freedom that exists for GET requests, where the actual URL parameter used to defined the next page 
of results is purely up to the implementor and not defined in the API spec, if the implementor decides to use headers, 
there are no specific or required header names defined in the specification.  Implementors may use any names or fields 
of their choosing. Pagination can be provided solely through header values, solely through body values, or through some 
combination.  

To avoid returning the entire original request body in a POST response which may be arbitrarily large, the  `merge` 
property can be specified. This indicates that the client must send the same post body that it sent in the original 
request, but with the specified headers/body values merged in. This allows servers to indicate what needs to change 
to get to the next page without mirroring the entire query structure back to the client.

See the [paging examples](examples.md#paging-examples) for additional insight.

## HTTP Request Methods and Content Types

STAC APIs follow the modern web API practices of using HTTP Request Methods ("verbs") and
the `Content-Type` header to drive behavior on resources ("nouns").
This section describes how these are used with the `/asset/search` endpoint.

### GET

**Required**: STAC's cross-collection `/asset/search` requires GET queries for all implementations, following OAFeat's precedent of 
making GET required (it only specifies GET so far). 

### POST

**Recommended** STAC `/asset/search` is strongly recommended to implement POST `Content-Type: application/json`, where the content body is a JSON 
object representing a query and filter, as defined in this document. 

It is recommended that clients use POST for querying (if the STAC API supports it), especially when using the 
`intersects` query parameter, for two reasons:

1. In practice, the allowed size for an HTTP GET request is significantly less than that allowed for a POST request, 
so if a large geometry is used in the query it may cause a GET request to fail.
2. The parameters for a GET request must be escaped properly, making it more difficult to construct when using JSON 
parameters (such as intersect, as well as additional filters from the query extension).

**STAC API extensions** allow for more sophisticated searching, such as the ability to sort, select which fields you want returned, and 
searching on specific Item properties.

#### PUT / PATCH / DELETE

The other HTTP verbs are not supported in STAC Asset Search. The [Transaction Extension](../ogcapi-features/extensions/transaction/README.md)
does implement them, for STAC and OAFeat implementations that want to enable writing and deleting items.

## Extensions

The extension follow those in Item Search. See [Item Search](https://github.com/radiantearth/stac-api-spec/blob/master/item-search/README.md) 
