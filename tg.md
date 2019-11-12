# Setting up an INSPIRE Download service based on the OGC API-Features standard

`Version: 0.1`
`Date: 2019-11-12`

## Table of Contents
* [Introduction](#introduction)
* [Glossary](#glossary)
* [Normative references](#normative-references)
* [Download Services based on OAPIF](#oapif-download)
    * Main principles
    * OAPIF Conformance classes
    * INSPIRE-specific requirements
* [Annex A: Abstract Test Suite](ats)
* Annex B
* Annex C
* Annex D

## Introduction
This document proposes a technical approach for implementing an INSPIRE download service based on the newly adopted [OGC API-Features standard](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html). The approach described here is not legally binding and shows one of many possible ways of implementing INSPIRE Download services.

### Why another download service TG for INSPIRE?
Several possible solutions for implementing Download services are already endorsed by the INSPIRE Maintenance and Implementation (MIG) group. [Technical guidelines documents](https://inspire.ec.europa.eu/Technical-Guidelines2/Network-Services/41) are made available that cover ATOM, WFS 2.0, WCS and SOS. While all of those approaches use the Web for providing access to geospatial data, they require xxx 

### OGC-API Features - a brief overview

- Very short overview of the main features and ideas behind SDW OGC-API Features  (OAPIF) and how it is different from the other OGC standards used for INSPIRE download services

## Glossary

- Web API - API using an architectural style that is founded on the technologies of the Web [DWBP](https://www.w3.org/TR/dwbp)
Note: Best Practice 24: Use Web Standards as the foundation of APIs in the W3C Data on the Web Best Practices provides more detail.

specific API implemented for a data set distribution. Standardisation target of OAPIF
- reuse relevant definitions from OAPIF, add INSPIRE-specific comments where necessary

## Normative references

- [OGC API - Features - Part 1: Core](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html)
- [OpenAPI 3.0](https://swagger.io/docs/specification/about)
- [IRs for NS]()
- [IRs for ISDSS]()
- [Data on the Web Best Practices](https://www.w3.org/TR/dwbp/)
- [RFC 7231 - Hypertext Transfer Protocol (HTTP/1.1)](https://tools.ietf.org/html/rfc7231)


## Download Services based on OAPIF
This section describes how to setup a Web API that is compliant with both ‘OGC API – Features’ and INSPIRE requirements for download services. 

The following conformance classes are defined:

1. INSPIRE-pre-defined-dataset-download-OAPIF
2. INSPIRE-direct-access-download-OAPIF (?)
3. INSPIRE-OAPIF-QoS (???)

### Main principles

- The landing page of the Web API `(path = /)` corresponds to a distribution of an INSPIRE data set
- The data set is structured into one or several collections --> all collections available in one API (under the `/collections` path) are considered to be part of the data set served by the Web API
- Every collection contains features of only one spatial object type
  - NOTE According to the OAPIF standard a collection could contain also more than one spatial object type.
 
EXAMPLE
2 data sets (with their own metadata records), one on buildings and one on addresses

http://my-org.eu/addresses/ - http://my-org.eu/addresses/api - http://my-org.eu/addresses/collections
http://my-org.eu/buildings/

NOT http://my-org.eu/oapif/ - http://my-org.eu/oapif/collections/addresses and http://my-org.eu/oapif/collections/buildings

### OAPIF conformance classes

REQ: The Web API SHALL comply with [OAPIF CC Core]().

TEST: rely on OAPIF CC Core ATS and CITE test.

NOTE CRS=WGS84 when returning collections and features through the API. Enclosure link for bulk download could still have a different CRS.

REC: The web API SHOULD apply with OAPIF CC JSON. [Make this an INSPIRE REQ?]

REC: The web API SHOULD apply with OAPIF CC OpenAPI. [Make this an INSPIRE REQ?]

FUTURE WORK: CC/EXTENSION Filter – might be needed for direct access download services

### INSPIRE-specific requirements

The web API shall support requests for all resources (/, /api, /collections, /collections/{name}/items, /collections/{name}/items/{id}) in a natural language through HTTP content negotiation.

REQ: The web API SHALL support the accept-language HTTP header (RFC 7231) in requests for all resources.

TEST: Send HTTP request including an accept-language HTTP header for all operations required by this specification and check that no error is returned (HTTP response code 200 OK).

REQ: For the landing page of the web API (/), the web API SHALL include the content-language HTTP header (RFC 7231) in the response.

TEST: Send HTTP request including an accept-language HTTP header to / and check that a resource is returned (HTTP response code 200 OK) and that the content-language parameter is present in the HTTP header of the response.

OPEN QUESTION: How to retrieve the supported languages from the web API? E.g. using "alternate" links on the landing page and all resources (with a lang parameter) or in the OpenAPI definition? --> TO DO

```json
{
  "title": "Buildings in Bonn",
  "description": "Access to data about buildings in the city of Bonn via a Web API that conforms to the OGC API Features specification.",
  "links": [
	{ "href": "http://data.example.org/",
  	"rel": "self", "type": "application/json", "title": "this document", **"lang": "en"** },
	{ "href": "http://data.example.org?lang=de",
  	"rel": "alternate", "type": "application/json", "title": "this document in German", **"lang": "de"** },
	{ "href": "http://data.example.org/api",
  	"rel": "service-desc", "type": "application/vnd.oai.openapi+json;version=3.0", "title": "the API definition" },
	{ "href": "http://data.example.org/api.html",
  	"rel": "service-doc", "type": "text/html", "title": "the API documentation" },
	{ "href": "http://data.example.org/conformance",
  	"rel": "conformance", "type": "application/json", "title": "OGC API conformance classes implemented by this server" },
	{ "href": "http://data.example.org/collections",
  	"rel": "data", "type": "application/json", "title": "Information about the feature collections" }
  ]
}
```

REC: For all other operations, оf the requested resources is available in the language specified in the accept-language HTTP header of the request, the web API SHOULD return the requested resource in the requested language, and specify the language in the content-language HTTP header (RFC 7231).

REQ: The response of the /collections operation SHALL include at least one [only one?] "enclosure" link that allows requesting a representation of the whole data set.

TEST: check whether the /collections resource includes at least one "enclosure" link and that a request to all of these links returns a resource (HTTP response code: 200).
MANUAL TEST: Check that the resource returned is a distribution of the whole data set. (Needed?)

----------------
Example 4. Feature collections response document

This feature collections example response in JSON is for a dataset with a single collection "buildings". It includes links to the features resource in all formats that are supported by the service (link relation type: "items").

Representations of the resource in other formats are referenced using link relation type "alternate".

An additional link is to a GML application schema for the dataset - using link relation type "describedBy".

Finally there are also links to the license information for the building data (using link relation type "license").

Reference system information is not provided as the service provides geometries only in the default systems (spatial: WGS 84 longitude/latitude; temporal: Gregorian calendar).

```json
{
  "links": [
	{ "href": "http://data.example.org/collections.json",
  	"rel": "self", "type": "application/json", "title": "this document" },
	{ "href": "http://data.example.org/collections.html",
  	"rel": "alternate", "type": "text/html", "title": "this document as HTML" },
	{ "href": "http://schemas.example.org/1.0/buildings.xsd",
  	"rel": "describedBy", "type": "application/xml", "title": "GML application schema for Acme Corporation building data" },
	{ "href": "http://download.example.org/buildings.gpkg",
  	"rel": "enclosure", "type": "application/geopackage+sqlite3", "title": "Bulk download (GeoPackage)", "length": 472546 }
  ],
  "collections": [
	{
  	"id": "buildings",
  	"title": "Buildings",
  	"description": "Buildings in the city of Bonn.",
  	"extent": {
    	"spatial": {
      	"bbox": [ [ 7.01, 50.63, 7.22, 50.78 ] ]
    	},
    	"temporal": {
      	"interval": [ [ "2010-02-15T12:34:56Z", null ] ]
    	}
  	},
  	"links": [
    	{ "href": "http://data.example.org/collections/buildings/items",
      	"rel": "items", "type": "application/geo+json",
      	"title": "Buildings" },
    	{ "href": "https://creativecommons.org/publicdomain/zero/1.0/",
      	"rel": "license", "type": "text/html",
      	"title": "CC0-1.0" },
    	{ "href": "https://creativecommons.org/publicdomain/zero/1.0/rdf",
      	"rel": "license", "type": "application/rdf+xml",
      	"title": "CC0-1.0" }
  	]
	}
  ]
}
```
-------------------

REQ: every collection SHALL contain features of only one spatial object type
  - NOTE According to the OAPI standard a collection could contain also more than one spatial object type.
 
MANUAL TEST: Check for every collection, that all its spatial objects belong to the same spatial object type.

REQ: For each collections that provides data that has been harmonised according to the IRs on data interoperability, the name of the the collection SHALL be the  language-neutral name of the spatial object type as specified in the IR-ISDSS.

TEST: Check all collection names. If they have a recognised language neutral name, ok. If not -- MANUAL TEST: Check that these collections have not yet been harmonised.

Reinforce OAPIF REC on providing the  licence link - Recommendation 10 /rec/core/fc-md-license and REC to include a licence for the API in the Open API spec.



## Annex A: Abstract Test Suite

- could be done by reference to a repository in the INSPIRE validation Github space


## Annex B: Introduction to OGC-API Features

- Do we need this? Reference to Github repo…

https://github.com/opengeospatial/ogcapi-features


## Annex C: Mapping the requirements from the IRs to the OGC-API Features standard (and extensions)
| IRs for Network Services | OAPIF equivalent|
| ------------- |:-------------:|
| col 3 is      | right-aligned |
| col 2 is      | centered      |
| zebra stripes | are neat      |



## Annex D: An example – link to github page


