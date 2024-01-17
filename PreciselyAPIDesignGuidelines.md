# Precisely API Design Guidelines

## Table of Contents
**~Introduction~
~Resource modeling~
~HTTP methods~
~HTTP status codes~
~Query Parameters~
~Response body~
~Error Responses~
~API Versioning~
~Pagination~
~Content negotiation~
~Compression~
~Caching~
~Statelessness~
~Idempotency~
~API Security~
~Date/Time~
~Open API Docs~**
 
# Introduction
Most modern web applications expose APIs that clients can use to interact with the application. A well-designed web API should aim to support:
**Platform independence:** Any client should be able to call the API, regardless of how the API is implemented internally. This requires using standard protocols and having a mechanism whereby the client and the web service can agree on the format of the data to exchange.
**Service evolution:** The web API should be able to evolve and add functionality independently from client applications. As the API evolves, existing client applications should continue to function without modification. All functionality should be discoverable so that client applications can fully use it.
The standard defined in this document provides guidance on things that you should consider when designing web APIs. The guidelines described below are based on industry wide practices, pragmatic REST principles and ease of use for developers. Based on these objectives, these standards and guidelines will attempt to provide APIs that appear logically connected, are self-documenting and easy for the developers to guess patterns and start using quickly. Private or internal APIs should also try to follow these guidelines because internal services tend to eventually be exposed publicly. Consistency is valuable to not only external customers but also internal service consumers, and these guidelines offer best practices useful for any service.
These guidelines aim to achieve the following:
* Define consistent practices and patterns for all API endpoints across Precisely.
* Adhere as closely as possible to accepted REST/HTTP best practices in the industry at-large. 
* Make accessing Precisely Services via REST interfaces easy for all application developers.
* Allow service developers to leverage the prior work of other services to implement, test and document REST endpoints defined consistently.
⠀ 
# Resource modeling
Providing a hierarchy of resources makes it easy for developers to discover services. It enables the developer to predict patterns and minimizes the need to read API documentation and makes the overall API usage easy. Base resources are generally nouns and not verbs. An example of a resource hierarchy 
 
![](PreciselyAPIDesignGuidelines/Screenshot%202024-01-12%20at%2014.33.11.png)
 
In the above example, GET /products returns a list of all the products.
GET /products/111 returns the details of a product with id 111.
 
A typical URI for a managed RESTful API MUST be represented as follows,
| ~[https://api.precisely.com/{product}/{version}/{resource}?queryParam1=abc&queryParam2=xyz](https://api.precisely.com/%7bproduct%7d/%7bversion%7d/%7bresource%7d?queryParam1=abc&queryParam2=xyz)~ |
**{product}**
Represents a product or a product family. May also represent capability that is not specific to a product and therefore be categorized as common services.
Some considerations for product are:
location-intelligence, saase2e, etc. 
As part of the resource modeling activity, SMEs from the respective domains will recommend appropriate products to be exposed.
**{version}**
API Versions are represented as V1, V2, etc. Any minor version must be backwards compatible and therefore does not require a change in version number.
**{resource}**
A Resource is an object or a representation of an entity that has some data associated with it. Examples of resources are, products, features, plans, etc.,
**{queryParam}**
Query parameters are optional URI elements and can be used to modify the behavior of the API.
 
**Developers must perform resource modeling as part of the API design process**
As a part of the design the API developers MUST perform resource modeling which will in turn help in defining the resources and the operations to be performed on them.
**Resource names must be nouns (not verbs) and should map to conceptual domain entities**
Resources MUST generally be nouns and not verbs and should map to conceptual domain entities wherever possible.
**URL Paths should contain plural form of resources/entities while the HTTP method should define the action to be performed**
URL paths SHOULD contain plural form of resources/entities and the HTTP method in the request SHOULD define the kind of action to be performed on the resource. E.g, if *product* is a resource then *products* should be the top-level path element. For instance, to create a new *product* the following URL path to be used POST /products.
**Establish and maintain a Hierarchy of resources**
While identifying sub-resources under a base resource it is recommended to establish and maintain a resource hierarchy. 
**Avoid very fine-grained APIs with respect to sub-resources**
Enough care should be taken while identifying the sub-resources such that the APIs are not too fine grained as it could result in client applications being forced to invoke multitude of APIs to accomplish a business operation. Thus, it lends to believe that every identified sub-resource should be helpful in accomplishing a specific business operation.
**Avoid using mixed-case resource and sub-resource names**
Based on ~[W3 specification](https://www.w3.org/TR/WD-html40-970708/htmlweb.html)~, URLs are generally considered to be case-sensitive. Hence it may be best to avoid camel-casing of the resource and sub-resource names so that the caller can avoid issues due may arise due to mixed casing. Whenever the resource names constitute multiple words it is recommended to use hyphen as the delimiter instead of camel casing . However, hyphens must be avoided for the field names in request or response body. ~[Camel](https://tools.ietf.org/html/rfc3986#section-6.2.2.1)~-Casing is best suited for that purpose. Also, Query parameters should be case insensitive, refer [~[RFC3986](https://tools.ietf.org/html/rfc3986)~].

# HTTP methods
HTTP methods operate on a collection or a resource. The HTTP protocol defines a number of methods that assign semantic meaning to a request. 

| HTTP Methods | Operation | Description |
|---|---|---|
|   <br>GET | On a collection<br>/products |  <br>Return a list of products. It should return a 200 response with an empty array (or array within a pagination type) when there are no resources to return. |
|   <br>GET |  <br>On a resource<br>/products/111 | Get details on products with id 111. Additional parameters required to perform the GET operation are passed as query parameters. |
|   <br>POST | On a collection<br>/products | Create a new product. The details of the record to be created is provided as part of the POST body.  Response should return an HTTP Location header that is the GET URL of the resource. |
|   <br>PUT | On a resource<br>/products/111 |  <br>Update the entire record 111 with the contents of the body. |
|   <br>PATCH | On a resource<br>/products/111 | Updates the record 111 only with the fields present in the body. (i.e. partial updates) |
|   <br>DELETE | On a resource<br>/products/111 |  <br>Deletes record 111. May also operate on a collection. |
|   <br>OPTIONS | On a resource or collection |  <br>Returns a list of supported HTTP methods |


|  |
|---|---|
|  |
![](PreciselyAPIDesignGuidelines/unknown.png)

# HTTP status codes
HTTP status codes are returned as part of all responses. The following is a list of the status codes commonly returned by the various operations:

| Status code | Description |
|---|---|
|   <br>200 OK | A generic successful response. Typically returned by a GET or a PUT returning information. |
| 201 CREATED | Returned by a POST method after a record is successfully created.  The create resource URI should be specified in the Location HTTP header or its resource URI should match the URI of the POST (see ~[RFC 7231](https://tools.ietf.org/html/rfc7231#section-6.3.2)~) |
| 202 ACCEPTED <br>  | This status code indicates that the request has been accepted by the server for processing, but the processing has not been completed yet.<br>  |
| 204 NO CONTENT <br>  | The server has successfully fulfilled the request and that there is no additional content to send in the response payload body.<br>  |
| 301 MOVED PERMANENTLY <br>  | The target resource has been assigned a new permanent URI and any future references to this resource ought to use one of the enclosed URIs. The server SHOULD generate a Location header field in the response containing a preferred URI reference for the new permanent URI.<br>  |
| 303 SEE OTHER <br>  | A 303 response to a GET request indicates that the server does not have a representation of the target resource. The Location field value in the response should refer to the resource on which a GET can be invoked to retrieve the target resource. |
|   <br>400 BAD REQUEST | These are returned by various methods when the data provided by the client (as part of URI, query parameters or the body) does not meet server requirements. |
| 401 UNAUTHORIZED | Authentication information not provided, or authentication failed |
|   <br>403 FORBIDDEN | Usually indicates that the authentication succeeded but the credentials do not have sufficient permissions to access resource |
|   <br>404 NOT FOUND |  <br>The requested resource is not found |
|   <br>405 METHOD NOT ALLOWED |  <br>The method requested is not allowed for the resource in the URI |
|   <br>406 NOT ACCEPTABLE | Requested media type (in the Accept header) is not supported. For e.g. supported media types include application/json and application/xml |
|  409 CONFLICT | The request could not be completed due to a conflict with the current state of the target resource. This code is used in situations where the caller might be able to resolve the conflict and resubmit the request. |
|  410 GONE | The target resource is no longer available at the origin server and that this condition is likely to be permanent. |
|  415 UNSUPPORTED MEDIA TYPE | The origin server is refusing to service the request because the payload is in a format not supported by this method on the target resource. |
|  429 TOO MANY REQUESTS | The user has sent too many requests in a given amount of time. To be used when throttling is performed on a per user/customer basis. |
|  500 INTERNAL SERVER ERROR | An internal error was encountered that prevents the server from processing the request and providing a valid response |
|   503 SERVICE UNAVAILABLE | The server is currently unable to handle the request due to a temporary overload or scheduled maintenance, which will likely be alleviated after some delay. The server MAY send a Retry-After header field~[1](https://httpstatuses.com/503#ref-1)~ to suggest an appropriate amount of time for the client to wait before retrying the request. |
| 504 Gateway Timeout | The server was acting as a gateway or proxy and did not receive a timely response from the upstream server. |
Additional HTTP status codes may be returned based on specific product requirements. 4XX errors are classified as client errors, and the API consumer can make appropriate changes to the request and receive a successful response. 5XX errors are classified as server errors and can be resolved only by the API provider.

# Query Parameters
Query parameters are used to provide optional inputs and modify the results of an operation. Below are some of the query parameters recommended by this document that can be utilized as applicable to the corresponding APIs:

| Parameter | Description |
|---|---|
| sortOrder | For a GET operation on a collection, this parameter can be used to specify the sort order. |
| offset | Starting key or index/offset into the collection in a paginated response. |
| limit | Parameter to control the number of records to be returned in the response. To be used for paginated API calls. Defaults to a API specific max size as defined by the server. |
| details | For a GET operation, the level of details returned can be specified using this parameter. The values are full, basic, compact, summary. The implementation is resource specific. |

### Other examples of query parameters are - searchBy, startDate, endDate, etc.
# Response body
For most methods a body is returned as part of the response. Standard responses are as follows.
| HTTP methods | Status code | Response body |
|---|---|---|
| GET | 200 OK | Information requested |
| POST (requesting information) | 200 OK | Information requested |
|   <br>POST (create new resource) |  <br>201 CREATED | URI to the newly created resource is returned in the Location<br>header |
| PUT | 200 OK | Optionally return Id to resource updated |
| PATCH | 200 OK | Optionally return Id to resource updated |
| DELETE | 200 OK | Optionally return Id of resource deleted |

# Error Responses
In case of errors, the API returns the appropriate status codes. The response body may also contain additional error information. Error responses can be returned as part the following status codes.
| Status code | Error Response |
|---|---|
| 400 BAD REQUEST | Required |
| 401 UNAUTHORIZED <br>403 FORBIDDEN <br>404 NOT FOUND <br>405 METHOD NOT ALLOWED <br>406 NOT ACCEPTABLE <br>409 CONFLICT <br>410 GONE <br>415 UNSUPPORTED MEDIA TYPE <br>429 TOO MANY REQUESTS <br>  | Optional<br>  |
|   <br>500 INTERNAL SERVER ERROR <br>503 SERVICE UNAVAILABLE <br>  |  <br>Error body that provides an error code that will help debug the server issue. Any other fields included in the error response MUST NOT provide any internal secure information that is private to Precisely~[JV9]~ ~[KD10]~ .<br>  |


The format for the error body should follow ~[RFC 7807](https://tools.ietf.org/html/rfc7807)~ “Problem Details”, an extensible error response format whose defined fields include:
.
| Element | Required | Description |
|---|---|---|
|   <br>type |  <br>No | A URI reference [~[RFC3986](https://tools.ietf.org/html/rfc3986)~] that identifies the<br>      problem type.  This specification encourages that, when<br>      dereferenced, it provide human-readable documentation for the<br>      problem type (e.g., using HTML [~[W3C.REC-html5-20141028](https://tools.ietf.org/html/rfc7807#ref-W3C.REC-html5-20141028)~]).  When<br>      this member is not present, its value is assumed to be<br>      "about:blank". |
| title <br>  | No |  A short, human-readable summary of the problem<br>      type.  It SHOULD NOT change from occurrence to occurrence of the<br>      problem, except for purposes of localization (e.g., using<br>      proactive content negotiation; see ~[\[RFC7231\], Section 3.4](https://tools.ietf.org/html/rfc7231#section-3.4)~). |
|   <br>status <br>  |  <br>No |  The HTTP status code (~[\[RFC7231\], Section 6](https://tools.ietf.org/html/rfc7231#section-6)~)<br>      generated by the origin server for this occurrence of the problem. |
| detail <br>  | No | A human-readable explanation specific to this<br>      occurrence of the problem. |
| Instance |   | A URI reference that identifies the specific<br>      occurrence of the problem.  It may or may not yield further<br>      information if dereferenced. |
 
See RFC 7807 for further examples and discussion of adding custom extension members/properties.
Error description fields should not provide any implementation details (like stack traces etc.,) or customer details that are considered to be secure.

# API Versioning
It is recommended that minor changes in API request or responses should be backwards compatible. A major change to the interface (or capability) must be provided as a new version. The managed API platform supports API versioning for major changes to the API via the use of version numbers in the URI. E.g:
 
GET https://api.precisely.com/{product}/V1/{resource}  
Changes that usually do not require a new version are as below,
·       New resources or sub-resources getting added
·       New Http Methods on existing resources
·       New data formats additions on the existing APIs like adding support for additional geometry formats like WKT.
·       New fields or elements on existing entities (request/responses)
 Changes that mandatorily require a new version are as below,
·       Changes to the URLs, request, responses (excluding addition of new fields).
·       Removal of a support of particular HTTP method
·       Modifying the data type or any characteristic of an existing field or parameter etc., e.g., making a field mandatory
·       Changing the semantics of the API.  Ex. Creating a workflow that creates a document and then sends an email is changed to have separate document creation and notification APIs.
To support updates to an entity using PATCH rather than a PUT method. PATCH method allows updates in a more backward compatible way than a PUT which expects all the fields (including any new fields that were added) to present. The semantics of PUT are that it completely replaces the state of the resource. This means that the client application must include all the properties of any resource they are updating, even properties that didn’t exist at the time the client application was written. This puts a burden on client applications and makes the server itself vulnerable to client errors. PATCH, on the other hand, only changes the data explicitly referenced by the client. The server takes responsibility for the merge, which is safer.
 
API developers must have a definitive strategy for deprecating older versions of API and eventual retirement of the same. Note that at any given point in time it is recommended to not have more than two older versions. When a newer API version is introduced, there must be a phase out plan published for the oldest version of the API.

## Pagination
APIs that return a collection of entities in the response should necessarily consider paginating the response to limit the number of such entities that are returned in a single response. This not only helps with efficient transfer of information to the client application but also protects the API implementation from pulling huge amounts of data from the storage etc., which can bring down the server.
Hence there are two forms of pagination that may be supported by the APIs. Server/API implementation driven paging mitigates against DoS kind of attacks by forcibly paginating a request over multiple response payloads. Client-driven paging enables clients to request only the number of resources that it can use at a given time.
 Server driven paging must indicate that the response includes only a partial set and the response body must include the following fields,
·       totalPages – total number of pages based on the given page size
·       totalItems – total number of records in the collection
·       currentPage - the current page number of results
·       pageSize – number of elements returned in the current page
 To support client driven paging the API must provide two query parameters in the request
~~·~~       page - the page number of results (and the first page should be page number 1)
·       limit – indicates the number of records per page that the client can accept in the response
 The API implementation SHOULD honor the values specified by the client; however, the API documentation must clearly note that the clients should be prepared to handle responses that contain a different page size.
The API implementation should fail the request if the page requested by the client does not exist. 
Both the above forms of paging will depend on the collection of items having a stable order i.e., the client (or the server) cannot change the order of the items in the collection midway during the traversal of the pages.
Between client requested paging and server driven paging the API should always default to the server-side page settings. Sorting and Filtering parameters MUST be consistent across pages, because both client- and server-side paging is fully compatible with both filtering and sorting.
 Filtering, sorting and pagination operations may be performed against a given collection all at the same time. When all these operations are applied by the client at the same time, the following should be the order of evaluation:
* Filtering – filter the records in the collection based on the provided filter criteria
* Sorting – sort them by a given or default order on a certain sort criteria
* Pagination – the materialized paginated view is presented over the filtered and sorted list of records of a collection. This should apply to both client driven as well as server driven pagination.

⠀Alternately, an indeterminate paging type (ex. Spring Slice) may utilize a structure like:
·      currentPage
·      pageSize
·      isFirst
·      isLast
### This may be useful in scenarios like infinite-scroll, where content is being added continuously (ex. Twitter), or for performance reasons since it avoids having to obtain totals.
## Content negotiation
All REST APIs must support JSON as the default content type for request and responses as applicable. Under certain conditions (legacy APIs etc.,) XML could be an alternative format.
 Under rare circumstances APIs may support both JSON and XML content types in which case the API implementation should support client application (agent) driven content negotiation using HTTP Headers (like Accept etc.,). Implementing the Accept header based content negotiation is the most commonly used and recommended way of content negotiation.
 If the client application is requesting for non-supported content type via the Accept header or if the client application did not include an Accept header in the request, then the API implementation should select JSON as the response body content type duly including the Content-Type header.
**Guideline:**  As per Accept header specification it is possible for the client application to include multiple values in Accept header. For example,
Accept: application/json,application/xml;q=0.9,*/*;q=0.8
Above Accept header indicates that the client application is requesting for a JSON format in the response. If the API cannot respond in JSON then API could return XML format (the second level). It also indicates that the client can accept any format as the last preference. The API implementation MAY adhere to such an algorithm.
 Content negotiation using URL patterns as given below are NOT recommended 
Examples of URL formats that are **NOT** recommended.:
https://api.precisely.com/sample-product/v1/customers/20423.xml
https://api.precisely.com/sample-product/v1/customers/20423.json

## Compression
For API implementations whose requests or responses could carry huge content it is recommended that such APIs utilize compression. To avoid excessive usage of the resources and inefficient transfer of request/response messages via the APIm Gateway network it is recommended that the request/response content is compressed wherever the size exceeds 1MB.
 API implementations should support Accept-Encoding header in the requests that says what kind of compression algorithms the client understands. APIs should support the most common compression representations of “gzip” and “compress”.
 If the client sends an Accept-Encoding header in the request with the encoding format that is not supported by the API then the server should respond with a 406 Not Acceptable status code.
Similarly, the API should support Content-Encoding header in the request and response that describes the compression algorithm applied on the request/response content. If the API does not support a particular content encoding format in the request, then it should respond with a 415 Unsupported Media Type to the caller.
The API documentation should explicitly recommend the client applications to utilize and request for compression and also the algorithms supported by the API. Compression of the content should occur between the client application and the backend API implementation or the web container that is processing the request/response. For such requests/responses APIm Gateway should not apply any transformation to the request or response body. APIm should only be used as a proxy in such situations.

## Caching
Being cacheable is one of the main tenets of RESTful APIs. GET requests should be cacheable by default, until certain specific conditions arise. POST requests should not be cacheable by default, however under certain circumstances can be made cacheable if either an Expires header or a Cache-Control header with a directive, to explicitly allow caching, is added to the response by the server. Responses to PUT and DELETE requests should NOT be cacheable at all.
 APIs can utilize the two main headers Expires and Cache-Control to indicate whether the response can be cached, if so by whom and for how long. Cacheable responses (whether to a GET or to a POST request) should also include a validator — either an ETag or a Last-Modified header.
 If a particular response should not be cached along the route to the client applications, the API implementation should explicitly indicate it via the Cache-Control: no-cache directive in the response.
Client applications should be able to request a non-cached version of the resource representation using the Cache-Control: no-cache in the request header and the API should adhere to the same. 

## Statelessness
As per REST principles API implementations do not store any state about the client application’s session on the server side. Each request from the client to the server should contain all the information necessary to process the request. Thus, the client is responsible for maintaining all the application state related information.
API developers should distinguish between Application State and Resource/entity State. Application state is any information that can be used to identify the incoming client requests, their previous interactions and current context information (e.g., cookie etc.,). Whereas Resource state is the current state of a resource/entity representation on the server. All REST APIs should NOT maintain any application state on the server side.
 The client should be responsible for sending any state information to the server whenever it is needed. The APIs should not create any session on the server and there should be no session affinity on server. This would allow requests to be routed to any server in the farm and hence make the API implementation scalable. The API implementations should ensure that every request MUST stand alone and should not be affected by the previous conversation happened from the same client in past.

## Idempotency
An idempotent HTTP request is a request that can be invoked multiple times without any difference in the outcome. Based on the REST principles all HTTP methods other than POST are idempotent.
Generally, POST requests are not idempotent. This means that two POST requests sent to a collection resource with exactly the same payload MAY lead to multiple items being created in that collection. This is often the case for insert operations on items with a server-side generated id. However, there is an exception to this principle especially to handle network timeout scenarios and other unexpected error conditions,
*For example:* A Client application issued a POST request and due to a network error condition or due to a intermediary server time out issue, the client application never saw a 201 response from the origin server. When the client recovers from that error and retries the exact same request then the server may respond with a failure indicating that the resource already exists, and the client will be forced to invoke a search query to retrieve the resource information or may be invoke such a query even before attempting the POST request again. This can be avoided.
For APIs wherein the create or insert (i.e., POST method) operation has a unique identifier (either the primary key or a composite key) that is provided by the client application then the implementation should ensure that the API behaves in an idempotent manner i.e., resource state should not change when multiple identical requests are received. Also, multiple such records should NOT be created on the server.
## API Security
API security should be an integral part of the API design. API Gateway is responsible for securing the APIs onboarded. However, the API developer is responsible for securing the API backend and developer can use IP Whitelisting and *Mutually authenticated TLS.*
 
## Date/Time
To avoid any interoperability problems between preferences from country to country, portable date time formats are recommended. It is recommended that the representations in request/response utilize the format as defined in RFC3339 for formatting the dates, times and date-time values. The RFC3339 formatted values primarily adheres to Coordinated Universal Time (UTC) thus avoiding issues related to time zones and daylight-saving time setting changes.
E.g., 2019-10-12T07:20:50.52Z 
The above format has the advantage of readability, ensures interoperability and also the clients can compare two values by sorting them as strings.
# Open API Docs
APIs published should have Open API Specification for API documentation. The Open API Specification contract describes what the API does, it’s request parameters and response objects, all without any indication of code implementation. Web services defined with Open API Specification can communicate with each other irrespective of the language they’re built in, since it is language agnostic and machine readable. 
 
Open API Specs Generation During Build time: Every API in DI should should incorporate this mechanism of ~[KP11]~ to generate the Open API Specification  based on the meta-data/ added against the various resources, methods and controllers which will be in the form of code annotations.. This meta-data will generate the contract, client-side code, and other artifacts during runtime.  The tools trigger as the various methods and functions are called against their resources and produces the Open API Docs from the metadata defined in the API. 
 
Springdoc-openapi and ~[Swagger-core](https://github.com/swagger-api/swagger-core)~  are the main Java implementation of Swagger which also supports JAX-RS and plain servlets, using these libraries in java you can generate the Open API specification based on the meta-data added in the form of code annotations.  Similarly, the two main OpenAPI implementations for .NET are ~[Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore)~ and ~[NSwag](https://github.com/RicoSuter/NSwag)

