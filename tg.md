# Setting up an INSPIRE Download service based on the OGC API-Features standard

`Version: 0.1`
`Date: 2019-11-12`

## Table of Contents
* [Introduction](#introduction)
* [Scope](#scope)
* [Conformance](#conformance)
* [Normative references](#normative-references)
* [Terms and definitions](#terms-and-definitions)
* [INSPIRE Download Services based on OAPIF](#inspire-download-services-based-on-oapif)
    * [Main principles](#main-principles)
    * OAPIF Conformance classes
    * INSPIRE-specific requirements
* [Bibliography](#bibliography)
* [Annex A: Abstract Test Suite](#abstract-test-suite)
* Annex B
* Annex C
* Annex D

## Introduction
This document proposes a technical approach for implementing an INSPIRE download service based on the newly adopted [OGC API-Features standard](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html). The approach described here illustrates an additional option for implementing the requirements set out in the [INSPIRE Implementing Rules for download services](http://data.europa.eu/eli/reg/2009/976/oj). 

### Why another download service TG for INSPIRE?
Several possible solutions for implementing download services are already endorsed by the INSPIRE Maintenance and Implementation (MIG) group. [Technical guidelines documents](https://inspire.ec.europa.eu/Technical-Guidelines2/Network-Services/41) are available that cover implementations based on ATOM, WFS 2.0, WCS and SOS. While all of these approaches use the Web for providing access to geospatial data, they are not considered developer friendly, and require substantial knowledge of the standard involved. In contrast, the rapid emergence of Web APIs provide a flexible and easily understandable means for access to data. The W3C Data on the Web Best Practices therefore recommends that data be exposed through Web APIs [DWBP Best Practice 23](https://www.w3.org/TR/dwbp/#accessAPIs)[DWBP Best Practice 24](https://www.w3.org/TR/dwbp/#APIHttpVerbs). 

### OGC API - Features - a brief overview

OGC API standards define modular API building blocks to spatially enable Web APIs in a consistent way. The OpenAPI specification is used to define the API building blocks.

The OGC API family of standards is organized by resource type. The Open API - Features standard specifies the fundamental API building blocks for interacting with features. The spatial data community uses the term 'feature' for things in the real world that are of interest.

OGC API - Features provides API building blocks to create, modify and query features on the Web. OGC API - Features is comprised of multiple parts, each of them is a separate standard. The "Core" part specifies the core capabilities and is restricted to fetching features where geometries are represented in the coordinate reference system WGS 84 with axis order longitude/latitude. Additional capabilities that address more advanced needs will be specified in additional parts. Examples include support for creating and modifying features, more complex data models, richer queries, additional coordinate reference systems, multiple datasets and collection hierarchies.

By default, every API implementing this standard will provide access to a single dataset. Rather than sharing the data as a complete dataset, the OGC API - Features standards offer direct, fine-grained access to the data at the feature (object) level.

The standard specifies discovery and query operations that are implemented using the HTTP GET method. Support for additional methods (in particular POST, PUT, DELETE, PATCH) will be specified in additional parts.

Discovery operations enable clients to interrogate the API to determine its capabilities and retrieve information about this distribution of the dataset, including the API definition and metadata about the feature collections provided by the API.

Query operations enable clients to retrieve features from the underlying data store based upon simple selection criteria, defined by the client.

The standard defines the resources listed in Table 1. 
Resource
Path
HTTP method
Document reference
Landing page
/
GET
7.2 API landing page
Conformance declaration
/conformance
GET
7.4 Declaration of conformance classes
Feature collections
/collections
GET
7.13 Feature collections
Feature collection
/collections/{collectionId}
GET
7.14 Feature collection
Features
/collections/{collectionId}/items
GET
7.15 Features
Feature
/collections/{collectionId}/items/{featureId}
GET
7.16 Feature

Implementations of OGC API Features are intended to support two different approaches how clients can use the API.

In the first approach, clients are implemented with knowledge about this standard and its resource types. The clients navigate the resources based on this knowledge and based on the responses provided by the API. The API definition may be used to determine details, e.g., on filter parameters, but this may not be necessary depending on the needs of the client. These are clients that are in general able to use multiple APIs as long as they implement OGC API Features.

The other approach targets developers that are not familiar with the OGC API standards, but want to interact with spatial data provided by an API that happens to implement OGC API Features. In this case the developer will study and use the API definition - typically an OpenAPI document - to understand the API and implement the code to interact with the API. This assumes familiarity with the API definition language and the related tooling, but it should not be necessary to study the OGC API standards.

For a description of the main differences between WFS 2.0 and OGC API - Features, see [this section in the Guide on OGC API - Features](https://github.com/opengeospatial/ogcapi-features/blob/master/guide/section_8_WFS_2_0_v_3_0.adoc)

## Scope
This document proposes a technical approach for implementing an INSPIRE download service based on the [OGC API-Features standard](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html). The approach described here is not legally binding and shows one of many possible ways of implementing INSPIRE Download services.

## Conformance
This specification defines three requirements classes:
1. INSPIRE-pre-defined-dataset-download-OAPIF
2. INSPIRE-direct-access-download-OAPIF (?)
3. INSPIRE-OAPIF-QoS (???)

The target of all requirements classes are “Web APIs”.

TODO Add sth about dependencies between the requirements classes.

Conformance with this specification shall be assessed using all the relevant conformance test cases specified in Annex A (normative) of this specification.
## Normative references

- [OGC API - Features - Part 1: Core](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html)
- [OpenAPI 3.0](https://swagger.io/docs/specification/about)
- [IRs for NS]()
- [IRs for ISDSS](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex:32010R1089)
- [RFC 7231 - Hypertext Transfer Protocol (HTTP/1.1)](https://tools.ietf.org/html/rfc7231)

## Terms and definitions
For the purposes of this document, the following terms and definitions apply.

ISO and the European Commission maintain terminological databases at the following addresses:
- ISO Online browsing platform: available at https://www.iso.org/obp
- INSPIRE glossary: available at http://inspire.ec.europa.eu/glossary

- Web API - API using an architectural style that is founded on the technologies of the Web [DWBP](https://www.w3.org/TR/dwbp)
Note: Best Practice 24: Use Web Standards as the foundation of APIs in the W3C Data on the Web Best Practices provides more detail.

specific API implemented for a data set distribution. Standardisation target of OAPIF
- reuse relevant definitions from OAPIF, add INSPIRE-specific comments where necessary
## INSPIRE Download Services based on OAPIF
This section describes the requirements a Web API must fulfill in order to be compliant with both ‘OGC API – Features’ and INSPIRE requirements for download services.
### Main principles

- The landing page of the Web API `(path = /)` corresponds to a distribution of an INSPIRE data set
- The data set is structured into one or several feature collections --> all feature collections available in one API (under the `/collections` path) are considered to be part of the data set provided by the Web API
- Every collection contains features of only one spatial object type
  - NOTE According to the OAPIF standard a collection could contain also more than one spatial object type.
 
EXAMPLE
2 data sets (with their own metadata records), one on buildings and one on addresses

http://my-org.eu/addresses/
http://my-org.eu/addresses/api
http://my-org.eu/addresses/collections

http://my-org.eu/buildings/
http://my-org.eu/buildings/api
http://my-org.eu/buildings/collections

NOT http://my-org.eu/oapif/ - http://my-org.eu/oapif/collections/addresses and http://my-org.eu/oapif/collections/buildings

### Requirements class “INSPIRE-pre-defined-dataset-download-OAPIF”

TODO insert table containing id, target type and dependencies

#### OAPIF requirements

**REQ001:** The Web API SHALL comply with requirements class http://www.opengis.net/spec/ogcapi-features-1/1.0/req/core.

TEST: rely on OAPIF CC Core ATS and CITE test.

NOTE CRS=WGS84 when returning collections and features through the API. Enclosure link for bulk download could still have a different CRS.

**REC001:** The Web API SHOULD comply with OAPIF RC JSON. [Make this an INSPIRE REQ?]

**REC002:** The Web API SHOULD comply with OAPIF RC OpenAPI. [Make this an INSPIRE REQ?]

FUTURE WORK: RC/EXTENSION Filter – might be needed for direct access download services

#### INSPIRE-specific requirements

The web API shall support requests for all resources (`/`, `/api`, `/collections`, `/collections/{name}/items`, `/collections/{name}/items/{id}`) in a natural language through HTTP content negotiation.

**REQ002:** The Web API SHALL support the `Accept-Language` HTTP header [RFC 7231](https://tools.ietf.org/html/rfc7231) in requests for all resources.

TEST: Send HTTP request including an Accept-Language HTTP header for all operations required by this specification and check that no error is returned (HTTP response code 200 OK).

**REQ003:** For the landing page of the web API (`/`), the web API SHALL include the Content-Language HTTP header [RFC 7231](https://tools.ietf.org/html/rfc7231) in the response.

TEST: Send HTTP request including an Accept-Language HTTP header to / and check that a resource is returned (HTTP response code 200 OK) and that the Content-Language parameter is present in the HTTP header of the response.

OPEN QUESTION: How to retrieve the supported languages from the Web API? E.g. using "alternate" links on the landing page and all resources (with a lang parameter) or in the OpenAPI definition? --> TO DO

```json
{
  "title": "Buildings in Bonn",
  "description": "Access to data about buildings in the city of Bonn via a Web API that conforms to the OGC API Features specification.",
  "links": [
	{ "href": "http://data.example.org/",
  	"rel": "self", "type": "application/json", "title": "this document", "lang": "en"},
	{ "href": "http://data.example.org?lang=de",
  	"rel": "alternate", "type": "application/json", "title": "this document in German", "lang": "de" },
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


**REC003:** For all other operations, the requested resources are available in the language specified in the `Accept-Language` HTTP header of the request, the Web API SHOULD return the requested resource in the requested language, and specify the language in the `Content-Language` HTTP header [RFC 7231](https://tools.ietf.org/html/rfc7231).

**REQ004:** The response of the `/collections` operation SHALL include at least one [only one?] "enclosure" link that allows requesting a representation of the whole data set.

TEST: check whether the `/collections` resource includes at least one `enclosure` link and that a request to each of these links returns a resource (HTTP response code: 200).
MANUAL TEST: Check that the resource returned is a distribution of the whole data set. (Needed?)

----------------
Example 4. Feature collections response document

This feature collections example response in JSON is for a dataset with a single collection "buildings". It includes links to the features resource in all formats that are supported by the service (link relation type: `items`).

Representations of the resource in other formats are referenced using link relation type `alternate`.

An additional link is to a GML application schema for the dataset - using link relation type `describedBy`.

Finally, there are also links to the license information for the building data (using link relation type `license`).

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

**REQ005:** every collection SHALL contain features of only one spatial object type
  - NOTE According to the OAPIF standard a collection could contain also more than one spatial object type.
 
MANUAL TEST: Check for every collection, that all its spatial objects belong to the same spatial object type.

**REQ006:** For each `collection` that provides data that has been harmonised according to the IRs on data interoperability, the name of the collection SHALL be the  language-neutral name of the spatial object type as specified in the [IR-ISDSS](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex:32010R1089).

TEST: Check all collection names. If they have a recognised language neutral name, ok. If not -- MANUAL TEST: Check that these collections have not yet been harmonised.

Reinforce OAPIF REC on providing the licence link - Recommendation 10 `/rec/core/fc-md-license` and REC to include a licence for the API in the Open API spec.

## Bibliography
- [W3C Data on the Web Best Practices](https://www.w3.org/TR/dwbp/)
- [W3C Spatial Data on the Web Best Practices](https://www.w3.org/TR/sdw-bp/)
- [INSPIRE UML-to-GeoJSON encoding rule](https://github.com/INSPIRE-MIF/2017.2/blob/master/GeoJSON/geojson-encoding-rule.md)

# Annexes
## Annex A: Abstract Test Suite

- could be done by reference to a repository in the INSPIRE validation Github space


## Annex B: Introduction to OGC-API Features

- Do we need this? Reference to Github repo…

https://github.com/opengeospatial/ogcapi-features


## Annex C: Mapping the requirements from the IRs to the OGC-API Features standard (and extensions)
### Predefined access download service

| Service operation | Mapping to OAPIF|
| ------------- |:-------------:|
| 1. Get Download Service Metadata | |
| 2. Get Spatial Dataset | |
| 3. Describe Spatial Dataset | |
| 4. Link Download Service | |
### Direct access download service

| Service operation | Mapping to OAPIF|
| ------------- |:-------------:|
| 1. Get Spatial Object | |
| 2. Describe Spatial Object Type | |
| 3. Link Download Service | |

## Annex D: An example – link to github page


