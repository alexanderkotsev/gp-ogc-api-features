# Good Practice – Setting up an INSPIRE Download service based on the OGC API-Features standard

## Introduction

###Rationale: why another download services in INSPIRE?

### OGC-API Features - a brief overview
- Very short overview of the main features and ideas behind SDW OGC-API Features and how it is different from the other OGC standards used for INSPIRE download services

## Glossary

- Web API - API using an architectural style that is founded on the technologies of the Web [DWBP]
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


## Setting up INSPIRE download services based on OAPIF
Aim of the section: How to set up a Web API compliant with OGC API – Features and INSPIRE requirements for download services

Conformance classes:

- INSPIRE-pre-defined-dataset-download-OAPIF
- INSPIRE-direct-access-download-OAPIF (?)
- INSPIRE-OAPIF-QoS (???)

### Main principles

- landing page of the web API (path = /) corresponds to a distribution of a data set
- the data set is structured into one or several collections --> all collections available in one API (under the /collections path) are considered to be part of the data set served by the API
- Every collection contains features of only one spatial object type
  - NOTE According to the OAPI standard a collection could contain also more than one spatial object type.
 
EXAMPLE
2 data sets (with their own metadata records), one on buildings and one on addresses

http://my-org.eu/addresses/ - http://my-org.eu/addresses/api - http://my-org.eu/addresses/collections
http://my-org.eu/buildings/

NOT http://my-org.eu/oapif/ - http://my-org.eu/oapif/collections/addresses and http://my-org.eu/oapif/collections/buildings

### Reference to OAPIF conformance classes

REQ: The web API SHALL comply with OAPIF CC Core.

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

REC: For all other operations, uf the requested resources is available in the language specified in the accept-language HTTP header of the request, the web API SHOULD return the requested resource in the requested langauge, and specify the language in the content-language HTTP header (RFC 7231).

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


## Annex C: Mapping the requirements from the IRs to the OGC-API Features standard (and extensions)


## Annex D: An example – link to github page
