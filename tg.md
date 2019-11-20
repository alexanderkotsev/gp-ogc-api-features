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
    * Requirements class “INSPIRE-pre-defined-dataset-download-OAPIF”
        * [OAPIF requirements](#oapif-requirements)
        * [INSPIRE-specific requirements](#inspire-specific-requirements)
    * Requirements class “INSPIRE-direct-access-download-OAPIF”
    * Requirements class “INSPIRE OAPIF JSON”
* [Bibliography](#bibliography)
* [Annex A: Abstract Test Suite](#abstract-test-suite)
* Annex B: Mapping the requirements from the IRs to the OGC-API Features standard (and extensions)
* Annex C: Examples

## Introduction
This document proposes a technical approach for implementing the requirements set out in the [INSPIRE Implementing Rules for download services](http://data.europa.eu/eli/reg/2009/976/oj) based on the newly adopted [OGC API-Features standard](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html). 

Several possible solutions for implementing download services are already endorsed by the INSPIRE Maintenance and Implementation (MIG) group. [Technical guidelines documents](https://inspire.ec.europa.eu/Technical-Guidelines2/Network-Services/41) are available that cover implementations based on ATOM, WFS 2.0, WCS and SOS. 

While all of these approaches use the Web for providing access to geospatial data, the new family of OGC API standards aim to be more developer friendly by requiring less up-front knowledge of the standard involved. The rapid emergence of Web APIs provide a flexible and easily understandable means for access to data. The W3C Data on the Web Best Practices therefore recommends that data be exposed through Web APIs [DWBP Best Practice 23](https://www.w3.org/TR/dwbp/#accessAPIs)[DWBP Best Practice 24](https://www.w3.org/TR/dwbp/#APIHttpVerbs). 
Therefore, this document describes an additional option for the implementation of INSPIRE download services.

### OGC API - Features - a brief overview

OGC API standards define modular API building blocks to spatially enable Web APIs in a consistent way. The [OpenAPI specification](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html#OpenAPI) is used to define the API building blocks.

OGC API - Features provides API building blocks to create, modify and query features on the Web. OGC API - Features is comprised of multiple parts, each of them is a separate standard. The ["Core"](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html) part specifies the core capabilities and is restricted to fetching features where geometries are represented in the coordinate reference system WGS 84 with axis order longitude/latitude. Additional capabilities that address more advanced needs will be specified in additional parts. 

By default, every API implementing this standard will provide access to a single dataset. Rather than sharing the data as a complete dataset, the OGC API - Features standards offer direct, fine-grained access to the data at the feature (object) level. Query operations enable clients to retrieve features from the underlying data store based upon simple selection criteria, defined by the client.

For further details about the standard, see the [OGC API - Features Github space](https://github.com/opengeospatial/ogcapi-features). 

For a description of the main differences between WFS 2.0 and OGC API - Features, see [this section in the Guide on OGC API - Features](https://github.com/opengeospatial/ogcapi-features/blob/master/guide/section_8_WFS_2_0_v_3_0.adoc).

## Scope
This document proposes a technical approach for implementing the requirements set out in the [INSPIRE Implementing Rules for download services](http://data.europa.eu/eli/reg/2009/976/oj) based on the [OGC API-Features standard](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html). 

The approach described here is not legally binding and shows one of several ways of implementing INSPIRE Download services.

## Conformance
This specification defines three requirements classes:
1. INSPIRE-pre-defined-dataset-download-OAPIF (mandatory)
2. INSPIRE-direct-access-download-OAPIF (?) (optional)
3. INSPIRE-OAPIF-QoS (???) (mandatory)
4. INSPIRE-OAPIF-JSON (optional)

The target of all requirements classes are “Web APIs”.

TODO Add sth about dependencies between the requirements classes.

Conformance with this specification shall be assessed using all the relevant conformance test cases specified in Annex A (normative) of this specification.
## Normative references

- [OGC API - Features - Part 1: Core](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html)
- [OpenAPI 3.0] OpenAPI specification v3.0 (https://swagger.io/docs/specification/about)
- [IRs for NS] Regulation … (http://data.europa.eu/eli/reg/2009/976/oj)
- [IRs for ISDSS](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex:32010R1089)
- [RFC 7231 - Hypertext Transfer Protocol (HTTP/1.1)](https://tools.ietf.org/html/rfc7231)
- [RFC 2616] https://www.w3.org/Protocols/rfc2616

## Terms and definitions
For the purposes of this document, the following terms and definitions apply.

ISO and the European Commission maintain terminological databases at the following addresses:
- ISO Online browsing platform: available at https://www.iso.org/obp
- INSPIRE glossary: available at http://inspire.ec.europa.eu/glossary

- Web API - API using an architectural style that is founded on the technologies of the Web [DWBP](https://www.w3.org/TR/dwbp)
Note: Best Practice 24: Use Web Standards as the foundation of APIs in the W3C Data on the Web Best Practices provides more detail.
- Feature - spatial object
- Feature collection
- data set [INSPIRE]
- distribution (of a data set) [DCAT]
- download service [INSPIRE]
- pre-defined data set download service (?) [INSPIRE]
- direct access download service (?) [INSPIRE]
- API [OpenAPI?]
- Content negotiation [RFC 7231]
- Encoding (rule) [ISO 19118?]
## INSPIRE Download Services based on OAPIF
This section describes the requirements a Web API must fulfill in order to be compliant with both ‘OGC API – Features’ and INSPIRE requirements for download services.
### Main principles

- The landing page of the Web API `(path = /)` corresponds to a distribution of one INSPIRE data set

EXAMPLE two data sets (with their own metadata records), one on buildings and one on addresses - two landing pages

http://my-org.eu/addresses/
http://my-org.eu/buildings/

NOT http://my-org.eu/oapif/ - http://my-org.eu/oapif/collections/addresses  and http://my-org.eu/oapif/collections/buildings 

- The data set is structured into one or several feature collections --> all feature collections available in one API (under the `/collections` path) are considered to be part of the data set provided by the Web API
- Every collection contains features of only one spatial object type
  - NOTE According to the OAPIF standard a collection could contain also more than one spatial object type.

| INSPIRE resources | OAPIF resource | Example path | Document reference(?) |
| ------------- | ------------- | ------------- |-------------: |
| Data set (distributions) | Landing page | http://my-org.eu/addresses/ <br> http://my-org.eu/buildings/ |7.2 API landing page |
| Data set description | Feature collections |
| Spatial object type | Feature collection | http://my-org.eu/addresses/collections/address | 7.14 Feature collection |
...

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
 
### Requirements class “INSPIRE-pre-defined-dataset-download-OAPIF”

TODO insert table containing id, target type and dependencies

http://inspire.ec.europa.eu/id/spec/oapif-download/1.0/req/{req-id}
http://inspire.ec.europa.eu/id/spec/oapif-download/1.0/rec/{rec-id}
#### OAPIF requirements

**REQ001:** The Web API SHALL comply with requirements class http://www.opengis.net/spec/ogcapi-features-1/1.0/req/core.

TEST: rely on OAPIF CC Core ATS and CITE test.

NOTE CRS=WGS84 when returning collections and features through the API. Enclosure link for bulk download could still have a different CRS.

**REQ002:** The Web API SHALL comply with OAPIF RC OpenAPI.

NOTE 1 In OAPIF standard, this is an option RC. This specification proposes to make it a mandatory requirement for INSPIRE in order to facilitate the development of client applications, and in particular adding support in the INSPIRE geoportal.

NOTE 2 There are plans to add additional RCs for other API description standards (or standard versions) in the future (e.g. for OpenAPI v3.1). When additional RCs become available, this specification will be reviewed and possibly revised to include these as additional options.


#### INSPIRE-specific requirements

The requirements from the [NS IR] to support requests in different natural languages is implemented in the web API through HTTP content negotiation.

**REQ003:** The Web API SHALL support the `Accept-Language` HTTP header [RFC 7231](https://tools.ietf.org/html/rfc7231) in requests for all resources.

TEST: Send HTTP request including an Accept-Language HTTP header for all operations required by this specification and check that no error is returned (HTTP response code 200 OK). Send a  request with Accept-Language including *; q=0.0 and check that the server returns HTTP 406 (Not Acceptable). See section 7.3 How to Implement Language Negotiation and 7.7 How to Handle Negotiation Failures in "RESTful Web services cookbook".

**REQ004:** For the landing page of the web API (`/`), the web API SHALL include the Content-Language HTTP header [RFC 7231](https://tools.ietf.org/html/rfc7231) in the response.

TEST: Send HTTP request including an Accept-Language HTTP header to / and check that a resource is returned (HTTP response code 200 OK) and that the Content-Language parameter is present in the HTTP header of the response.

To understand which languages are supported by the Web API, a client needs to send a HEAD request with the language it is interested in and “*; q=0.0”. The WebAPI should then respond with the requested language as Content-Language (if the language is supported) or a 406 error (if the language is not supported).

**REQ005** The Web API shall support HTTP HEAD requests for the landing page (/), /collections and /collections/{id} including `Accept-Language` HTTP headers.

OPEN QUESTION: Also include /api and /conformance paths here? 

OPEN QUESTION: Do you have any other proposals for how to implement the requirement for the download service to advertise the natural languages it supports?


**REC003:** For all other operations, the requested resources are available in the language specified in the `Accept-Language` HTTP header of the request, the Web API SHOULD return the requested resource in the requested language, and specify the language in the `Content-Language` HTTP header [RFC 7231](https://tools.ietf.org/html/rfc7231).

**REQ006:** The response of the `/collections` operation SHALL include at least one [only one?] "enclosure" link that allows requesting a representation of the whole data set.

TEST: check whether the `/collections` resource includes at least one `enclosure` link and that a request to each of these links returns a resource (HTTP response code: 200).
MANUAL TEST: Check that the resource returned is a distribution of the whole data set. (Needed?)

**REC004** The link(s) with the link relation type “enclosure” should include a property “length” with the size in bytes of the data set.

OPEN QUESTION Confirm that the semantics of the “length” property is “size in bytes” ().

EXAMPLE Feature collections response document

This feature collections example response in JSON is for a dataset with a single collection "buildings". It includes links to the features resource in all formats that are supported by the service (link relation type: `items`).

Representations of the resource in other formats are referenced using link relation type `alternate`.

An additional link is to a GML application schema for the dataset - using link relation type `describedBy`.

There are also links to the license information for the building data (using link relation type `license`).

Finally, the link with the link relation type `enclosure` provides a reference to another distribution of the data set as a GeoPackage download of the complete data set (pre-defined download). The length property includes the size (in bytes?) of the data set.

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
  	"rel": "enclosure", "type": "application/geopackage+sqlite3", "title": "Pre-defined data set download (GeoPackage)", "length": 472546 }
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

**REQ007:** Every collection SHALL contain features of only one spatial object type
 
NOTE According to the OAPIF standard a collection could contain also more than one spatial object type.
 
MANUAL TEST: Check for every collection, that all its spatial objects belong to the same spatial object type.

**REQ008:** For each `collection` that provides data that has been harmonised according to the IRs on data interoperability, the name of the collection SHALL be the  language-neutral name of the spatial object type as specified in the [IR-ISDSS](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex:32010R1089).

TEST: Check all collection names. If they have a recognised language neutral name, ok. If not -- MANUAL TEST: Check that these collections have not yet been harmonised.

Reinforce OAPIF REC on providing the licence link - Recommendation 10 `/rec/core/fc-md-license` and REC to include a licence for the API in the Open API spec.

### Requirements class “INSPIRE-direct-access-download-OAPIF”

FUTURE WORK: RC/EXTENSION Filter – might be needed for direct access download services

### Requirements class “INSPIRE OAPIF JSON”

This RC is relevant when using a (Geo-)JSON encoding (e.g. those developed in 2017.2 for Addresses and Environmental Monitoring Features).

**REQ:** The Web API SHALL comply with OAPIF RC JSON.
## Bibliography
- [W3C Data on the Web Best Practices](https://www.w3.org/TR/dwbp/)
- [W3C Spatial Data on the Web Best Practices](https://www.w3.org/TR/sdw-bp/)
- [INSPIRE UML-to-GeoJSON encoding rule](https://github.com/INSPIRE-MIF/2017.2/blob/master/GeoJSON/geojson-encoding-rule.md)

# Annex A: Abstract Test Suite

## Conformance class “INSPIRE-pre-defined-dataset-download-OAPIF”

### Title of the requirement / test

**Purpose**: Test requirement x

**Prerequisites**:

**Test method**: 

* Check that the [ExtendedCapabilities](#ExtendedCapabilities) exists. If it does
  * Validate the [ExtendedCapabilities](#ExtendedCapabilities) element and its children according to the XML Schema definition for the WFS INSPIRE ExtendedCapabilities as defined in the http://inspire.ec.europa.eu/schemas/inspire_dls/1.0/inspire_dls.xsd
  * If the validation passes, pass the test.
* If no ExtendedCapabilities were found or validation fails, fail the test.

**Test type**: Automated / Manual

**Notes**:

# Annex B: Mapping the requirements from the IRs to the OGC-API Features standard (and extensions)
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

# Annex C: Mapping between INSPIRE NS Metadata elements and OpenAPI definition fields
### Network Services metadata

| INSPIRE NS Metadata element | OpenAPI field names |
| ------------ | ------------ |
Resource Title (M) | info/title
Resource Abstract (M) | info/description
Resource Type (M) |
Resource Locator (C) | 
Coupled Resource ©
Spatial Data Service Type (M) 
Keyword (M) | 
Geographic Bounding Box (M)
Temporal Reference (M)
Spatial Resolution (C)
Conformity* (M)
Conditions for Access and Use (M) | info/termsOfService or info/license
Limitations on Public Access (M) | info/termsOfService or info/license
Responsible Organisation (M) info/contact/name
Metadata Point of Contact (M) | info/contact/name
Metadata Date (M) 
Metadata Language (M) 
Unique Resource Identifier (M)

# Annex C: Examples
Link to github page
