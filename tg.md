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

- [OGC API - Features - Part 1: Core] (http://docs.opengeospatial.org/is/17-069r3/17-069r3.html)
- [OpenAPI 3.0] OpenAPI specification v3.0 (https://swagger.io/docs/specification/about)
- [IRs for NS] Regulation 976/2009 implementing Directive 2007/2/EC as regards the Network Services (https://eur-lex.europa.eu/eli/reg/2009/976/oj)
- [IRs for ISDSS] Regulation 1089/2010 implementing Directive 2007/2/EC as regards interoperability of spatial data sets and services (https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex:32010R1089)
- [RFC 7231] - Hypertext Transfer Protocol (HTTP/1.1) (https://tools.ietf.org/html/rfc7231)
- [RFC 2616] https://www.w3.org/Protocols/rfc2616
- [RFC 4647] PHILLIPS, A. and DAVID, M. (eds.). Matching of Language Tags [online]. Internet Engineering Task Force, September 2006. RFC 4647. Available from: https://tools.ietf.org/html/rfc4647
- [RSS 2.0] RSS 2.0 Specification (http://www.rssboard.org/rss-draft-1)

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
## Symbols and abbreviated terms
API	Application Programming Interface
DCAT	Data Catalog Vocabulary
JSON	JavaScript Object Notation
OAPIF	OGC API - Features
URL	Uniform Resource Locator
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

| Requirements class | http://inspire.ec.europa.eu/id/spec/oapif-download/1.0 |
| --- | --- |
| Target type | Web API |
| Dependency | OPAIF Requirements class "Core" |
| Dependency | RFC 7231 (Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content) |


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
##### Internationalization: request and response language

The requirements from the [NS IR] to support requests in different natural languages are met in a Web API through HTTP language negotiation, using HTTP headers as specified in [RFC 7231] and language tags as specified in [RFC 4647].

In addition, support for different natural languages for bulk download is met as described in section “Download of the whole dataset”.

**REQ003:** The Web API SHALL support the `Accept-Language` header in requests to the landing page (/), /collections, /collections/{collectionId}, /collections/{collectionId}/items and /collections/{collectionId}/items/{featureId} in accordance with [RFC 7231] and [RFC 4647]. The Web API SHALL return HTTP status code 406 when an Accept-Language HTTP header set to `*;q=0.0` is sent.

TEST:
Issue an HTTP GET request with an Accept-Language HTTP header containing a valid language tag to the following URLs: {root}/ and {root}/collections. For every feature collection identified in the response of {root}/collections, issue an HTTP request with an Accept-Language HTTP header containing a valid language tag to {root}/collections/{collectionId}, {root}/collections/{collectionId}/items. TODO
For every response, validate that the HTTP status code is 200.
Issue an HTTP GET request with the Accept-Language header set to `*;q=0.0` to the following URLs: {root}/ and {root}/collections. For every feature collection identified in step 1, issue an HTTP GET request with the Accept-Language header set to `*;q=0.0` to {root}/collections/{collectionId} and {root}/collections/{collectionId}/items.
For every response, validate that the HTTP status code is 406.


**REQ004:** The Web API SHALL include the `Content-Language` HTTP header in the response for a request to its landing page (/).

TEST:
Issue an HTTP GET request with an Accept-Language HTTP header containing a valid language tag to URL {root}/.
Validate that a response is returned with a `Content-Language` HTTP header.
Issue an HTTP GET request without a Accept-Language HTTP header to URL {root}/.
Validate that a response is returned with a `Content-Language` HTTP header.

**REC00x:** The Web API SHOULD take the language specified in the `Accept-Language` HTTP header of a request to all paths into account. The Web API SHOULD include the `Content-Language` HTTP header in the response for a request to all paths.

##### Internationalization: supported languages

[RFC 2616] does not define what such a response body exactly should look like, see also Annex B, and no other existing specifications have been identified that define this. As one of the principles in this specification is not to have any INSPIRE-specific extensions or requirements, this specification therefore does not give a stronger recommendation than REC00x. This specification may be updated when the response body returned with HTTP status code 406 is standardised.

**REC00x:** The Web API SHOULD return a response with a response body containing a list of all supported languages for requests with the Accept-Language HTTP header set to `*;q=0.0` to the landing page (/), /collections, /collections/{collectionId}, /collections/{collectionId}/items and /collections/{collectionId}/items/{featureId}.

When HTML is acceptable for the client and the Web API conforms to OAPIF Conformance Class HTML, this could look as follows:

```
# Request
GET {root}/ HTTP/1.1
Accept: text/html
Accept-Language: *;q=0.0
```

```
# Response
406 Not Acceptable
Content-Type: application/html
Content-Language: en

<html>
  <head>
    <title>Supported languages</title>
  </head>
  <body>
     <p>This server supports the following languages: English, Danish.</p>
  </body>
</html>
```

As long as the response body supplied in a machine-readable format, such as JSON, sent with HTTP status code 406, is not standardised, it is not possible to build applications, such as the INSPIRE Geoportal, that programmatically can access a list of supported languages and display that information to an end user.

A workaround for this is the following procedure:
Retrieve a list of languages of interest
For a EU-wide application, this could be a list of all EU official languages.
For another application, this could be a list of languages chosen by an end user.
For every all language of interest, issue an HTTP HEAD request to the landing page (/) having HTTP header Accept-Language set to `language_of_interest_tag,*;q=0.0` (e.g. `en,*;q=0.0`).
For every response: if the HTTP status code of the response is 200, add the language specified in Content-Language to the list of supported languages.

This workaround presumes that the following requirements and recommendations are followed:
Recommendation 2 of OGC API - Features - Part 1, regarding the support of HTTP method HEAD
REQ00x and REC00x regarding support for Accept-Language and Content-Languages


**REQ005** The Web API shall support HTTP HEAD requests for the landing page (/), /collections and /collections/{id} including `Accept-Language` HTTP headers.

OPEN QUESTION: Also include /api and /conformance paths here? 

OPEN QUESTION: Do you have any other proposals for how to implement the requirement for the download service to advertise the natural languages it supports?


##### Download of the whole data set

**REQ00x:** The response of the `/collections` operation SHALL include at least one [only one?] "enclosure" link that allows requesting a representation of the whole data set.

TEST:

Issue an HTTP GET request to {root}/collections.
Validate that at least one of the links returned in the response has `rel` link parameter `enclosure` (REQ00x).
For each of the links returned in the response having a `rel` link parameter equal to `enclosure`, issue an HTTP HEAD request to the path given in the `href` link parameter of that link.
For each of the responses:
If the HTTP status code is 405 (Method Not Allowed), the test verdict is inclusive.
If HTTP status code 200 is returned, the test verdict is “pass”.
Otherwise, the test verdict is “fail”.


MANUAL TEST: Check that the resource returned is a distribution of the whole data set. (Needed?)

**REQ00x** A link with the link relation type “enclosure” SHALL include the `type` link parameter containing a media type valid that is valid according to [RFC 6838].

TEST:

Issue an HTTP GET request to {root}/collections.
For each of the links returned in the response having a `rel` link parameter equal to `enclosure`, validate that the `type` parameter is present. TODO

**REQ00x** A link with the link relation type “enclosure” SHALL include the `hreflang` link parameter containing the language of that distribution. The value of `hreflang` SHALL follow [RFC 4647].

TEST:

Issue an HTTP GET request to {root}/collections.
For each of the links returned in the response having a `rel` link parameter equal to `enclosure`, validate that the `hreflang` parameter is present. TODO

**REC00x** A link with the link relation type “enclosure” SHOULD include the `length` link parameter containing the length in bytes.

OPEN QUESTION Confirm that the semantics of the “length” property is “size in bytes” ().

**REC00x** The link(s) with the link relation type “enclosure” SHOULD include the `title` link parameter containing a human-friendly name.



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
##### Organisation of a dataset in feature collections

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
- [Alla] ALLAMARAJU, Subbu. RESTful Web services cookbook. O’Reilly Media, 2010. ISBN 978-0-596-80168-7.
- [SO1] [How to properly send 406 status code?](https://stackoverflow.com/questions/4422980/how-to-properly-send-406-status-code)
- [SO2] [Format for 406 Not Acceptable payload?](https://stackoverflow.com/questions/50102277/format-for-406-not-acceptable-payload)

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
## Predefined access download service

| Service operation | Mapping to OAPIF|
| ------------- |:-------------:|
| 1. Get Download Service Metadata | |
| 2. Get Spatial Dataset | |
| 3. Describe Spatial Dataset | |
| 4. Link Download Service | |
### Supported languages
According to [RFC 2616]:

>   The 406 (Not Acceptable) status code indicates that the target resource does not have a current representation that would be acceptable to the user agent, according to the proactive negotiation header fields received in the request (Section 5.3), and the server is unwilling to supply a default representation.

>  The server SHOULD generate a payload containing a list of available representation characteristics and corresponding resource identifiers from which the user or user agent can choose the one most appropriate.  A user agent MAY automatically select the most appropriate choice from that list.  However, this specification does not define any standard for such automatic selection, as described in Section 6.4.1.

> [...]

> A specific format for automatic selection is not defined by this specification because HTTP tries to remain orthogonal to the definition of its payloads.  In practice, the representation is provided in some easily parsed format believed to be acceptable to the user agent, as determined by shared design or content negotiation, or in some commonly accepted hypertext format.


Similar to the example on to [SO2], this could look as follows when JSON is acceptable for the client:

```
# Request
GET {root}/ HTTP/1.1
Accept: application/json
Accept-Language: *;q=0.0
```

```
# Response
406 Not Acceptable
Content-Type: application/json

{
  { "acceptable" : [ "en", "da" ] }
}
```

[RFC 7807] defines a "problem detail" as a way to carry machine-readable details of errors in a HTTP response to avoid the need to define new error response formats for HTTP APIs.

If [RFC 7808] is to be used, the problem details object would have to be extended with additional members.

```
# Request
GET {root}/ HTTP/1.1
Accept: application/json
Accept-Language: *;q=0.0
```

```
# Response
406 Not Acceptable
Content-Type: application/problem+json
Content-Language: en

{
    "type": "https://example.com/to/be/decided",
    "title": "Not acceptable",
    "detail": "This server supports the following languages: English, Danish.",
    "instance": "/collections",
    "acceptable_languages": [ "en", "da" ]
   }

```

In short: there is currently no standard way to supply a list of supported languages.

The parts in this specification regarding the 406 HTTP status code are inspired by discussions on Stack Overflow ([SO1], [SO2]) and on chapter 7 in [Alla].

## Direct access download service

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
Coupled Resource (C) | 
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

--- 
**NOTE**

Additional metadata elements can be added to an OpenAPI definition through [extensions](https://swagger.io/docs/specification/openapi-extensions/), implemented through the introduction of fields beginning with `x-`. However, in order to streamline the implementation of metadata, this document does not propose any INSPIRE-specific extensions. 

--- 
# Annex C: Examples
Link to github page
