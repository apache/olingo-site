Title: Java Library for OData Version 4

Java Library for OData Version 4
================================

[TOC]

Overview
--------

The Open Data Protocol (OData) is a web-based protocol for querying and
updating data. It has been defined initially by Microsoft but is now an
[OASIS standard](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=odata).
This allows anyone to freely interoperate with OData implementations. Data
exposed via the OData standard can be consumed in any environment offering
HTTP-based connectivity. In addition, there are client SDKs available for
various platforms such as .Net, Java, PHP, JavaScript, etc.

For the Java platform, the [Apache Olingo project](http://olingo.apache.org/)
offers a library useful for implementing an OData service. It provides
services such as URL parsing, input validation, (de-)serialization of
content, request dispatching, etc., according to the OData specification.

The main parts of an OData service implementation are the metadata
definition, the run-time processing of requests, and the definition of the
web infrastructure. These parts will now be described in more detail.


Entity Data Model (Metadata)
----------------------------

The Entity Data Model (EDM) is the underlying metadata model of the OData
protocol. Within the EDM the following (main) elements are described:

  * Schemas
  * Entity Types
  * Complex Types
  * Type Definitions
  * Enum Types
  * Properties
  * Navigation Properties
  * Actions
  * Functions
  * Entity Container
    * Entity Sets
    * Singletons
    * Action Imports
    * Function Imports

A proper OData Service requires a valid and consistent EDM. In order to speed
up performance, the OData Library does no validation of the EDM.

The standard way of defining the metadata is to write code in so-called
EDM provider classes. It would be possible to add other sources for
metadata, e.g., a predefined metadata document.

EDM provider classes separate the run-time EDM object instances from code
that defines OData EDM objects. All provider classes simply define string
values for the different EDM elements. At run-time, objects are created only
as far as necessary. Since the typical request (apart from the request for
the metadata document, of course) can be processed with only the directly
involved EDM objects, this can be a significant performance improvement.
Furthermore, this solves the problem that many EDM objects depend on each
other, making it no easy task to find the right order of object
instantiation.

Service implementations can derive from `CsdlAbstractEdmProvider` and
override only those methods that should provide EDM objects. Almost all
provider methods have a parameter of type `FullQualifiedName` that specifies
for which (namespace-qualified) name a provider-object instance is to be
returned. In addition, there are some methods with the purpose of retrieving
a list of all names of a given object type; these are used only for the
output of the metadata document.


Run-Time Processing of Requests
-------------------------------

### General Setup ###

#### Design Considerations ####

The OData standard describes many different types of requests that have to be
answered by an OData service implementation. It would not be useful to have a
single processing interface or even a single method that a service
implementation has to implement. Different designs are possible how the
multitude of requests can be split into more manageable parts.

This library has been designed to handle a given OData request in a single
call to a processing method, including the complete query-options part. Which
method the request dispatcher calls is decided according to the HTTP method
and the representation type of the expected response. The HTTP method
describes the fundamental type of operation: `GET` for read requests, `PUT`
or `PATCH` for update requests, `POST` for create requests, and `DELETE` for
deletion requests. The representation type describes which information can be
retrieved from the OData service: an Entity representation, or only a link
URL, or even just a simple number as a count of entities.

The fact that a single method call occurs for a given request, as complicated
as it may be, leads immediately to the consequence that almost all processing
methods must be prepared to handle things like navigation and system query
options. Navigation in turn means that even if the response in the end may be
a simple count, the implementation may have to read entity collections, their
relations, and even function imports or bound functions that may occur before
the final `$count` in the request URI. Services therefore have to be
implemented carefully in order to respond to requests they are not prepared
to handle with a `Not Implemented` response and its corresponding HTTP status
code of 501.

A major advantage of the one-step approach is that the processing interfaces
don't have to make any assumptions how application data is to be transported.
The serialization and deserialization helper methods of the library of course
use library-defined data objects but service implementations are free to use
their own code for that tasks and still can use the core run-time library
functionality.

Many implementations will re-use large parts of code for essentially the same
requests that differ only in the returned representation. But there might be
optimizations: An SQL request could be much more efficient for count
determination if not all entities are retrieved first. Since this library does
not favor a specific implementation, the interface is designed to allow the
necessary flexibility.

#### Overall Handling ####

Processor classes have to be registered in order to be called from the
library core at run-time. The core run-time registers the `DefaultProcessor`
API class with default implementations of `MetadataProcessor`,
`ServiceDocumentProcessor`, and `ErrorProcessor`, but this can be overridden
simply by registering an own implementation.

Each class can implement one or more of the processor interfaces; all
processing methods have unique names across all processor interfaces.

Since the general design of the library is to have as little constraints as
possible, as little as possible happens automatically. The service developer
is responsible for the correct response to a given request. There are several
helper methods to ease this task considerably. But if you are not satisfied
with them, it is always possible to use your own implementation, even if that
does not conform to the OData standard.

#### Processor Interfaces and Processing Methods ####

All processor interfaces derive from the `Processor` interface which defines
a single `init()` method allowing to get a run-time instance of the `OData`
object and the `ServiceMetadata`. The `OData` object is the root object for
serving factory tasks and the single instance connecting the API interfaces
with their implementations; each thread (processing of one request) should
keep its own instance. The `ServiceMetadata` instance contains the Entity
Data Model and some other objects related to metadata.

All processing methods have at least `ODataRequest` and `ODataResponse`
objects as parameters. They are supposed to find all necessary information
about the request body and its headers in the request object and set the
corresponding information in the response object.

Processing methods that are not called with a static URI like `$batch`
additionally have a `UriInfo` parameter describing the request URI.

Processing methods that have to read request-body content additionally have a
`ContentType` parameter describing the request-body format in order to select
the correct deserializer.

Processing methods that have to deliver content in the response body
additionally have a `ContentType` parameter describing the requested format
in order to select the correct serializer.

An example with all parameters mentioned above is the method `createEntity()`
in the interface `EntityProcessor`:


~~~java
void createEntity(ODataRequest request, ODataResponse response,
    UriInfo uriInfo, ContentType requestFormat, ContentType responseFormat)
    throws ODataApplicationException, ODataLibraryException;
~~~

### Content Negotiation ###

#### Request Content Type ####

Requests that send data must provide a `Content-Type` HTTP header.
This is checked in the core run-time code. For each representation type
there is a list of content types that are supported by the implementation.

The method `modifySupportedContentTypes()` of the interface
`CustomContentTypeSupport` can be implemented to change that; it is even
possible to remove library-supported content types. Any implementation of
this interface has to be registered in the same way as processor classes in
order to have any effect.

Request content types that do not match the supported content types are
rejected with error messages and the HTTP status code 406 (Not Acceptable).

#### Response Content Type ####

A request for a response with a content type not matching the supported
content types is rejected with an error message and the HTTP status code 415
(Unsupported Media Type).

The list of supported content types can be modified as described for the
request content type. It is not possible to have different support for
responses than for requests.

### Exceptions ###

All processing methods can and should throw exceptions if something went
wrong.

Almost all library utility methods declare exceptions that derive from
`ODataLibraryException`. Those exceptions are handled by the core run-time
code which takes care of setting the correct response status code and error
text.

If a service implementation wants to signal an error itself, it can throw an
`ODataApplicationException`. The constructor of this exception allows to
define the response status code and the error text. Please note that the
OData standard mandates specific status codes in many places; this is not
enforced by the library.

If a `RuntimeException` occurs, the core run-time code sets the response
status code to 500 (Internal Server Error) and delivers a corresponding error
text.

### Response Status Code ###

Service implementations have to set the response status code explicitly.
Failing to do so results in a status code of 500 (Internal Server Error).
Please note that the OData standard mandates specific status codes in many
places; this is not enforced by the library.

### Helper Methods ###

#### Serialization ####

##### Serializers Dependent on Content Negotiation #####

For some representation types the OData standard defines different
representations, e.g., JSON and XML. For the JSON format there are different
sub-formats differing in the amount of metadata that is part of the content.

In order to have a uniform interface and to relieve the service
implementations from many switch statements, it is possible to get the
correct serializer from the `OData` object's `createSerializer()` method that
has a content type as parameter. The requested response content type is
passed as parameter to all processing methods that are supposed to create
response content.

The `ODataSerializer` interface has methods to serialize the following types
of content:

  * service document
  * metadata document
  * error document
  * entity
  * entity collection
  * primitive value
  * collection of primitive values
  * complex value
  * collection of complex values
  * reference to an entity
  * collection of references to entities

Every data serializer gets its run-time data as an instance of a type defined
in the `commons.api.data` package. It has also an additional parameter to
pass options like the context URL or expand settings in a single object.
Please note that according to the OData standard the context URL is mandatory
for service responses except for responses with no metadata at all.

##### Fixed-Format Serializers #####

Fixed-format serializers can be used for formats defined in the OData
standard that are not subject to content negotiation. The
`FixedFormatSerializer` instance to be got from the `OData` object's
`createFixedFormatSerializer()` method has methods for serializing a binary,
a count, a primitive raw value, a batch, and an asynchronous response.

#### Deserialization ####

Deserialization works along the same principles as serialization, with
minor differences, however.

There is no deserializer for the representation types service document,
metadata document, and error document. There is an additional representation
type that can be deserialized: action parameters.

All content-type-dependent methods don't return data objects directly but
instances of `DeserializerResult` instead. This makes it possible to return
additional information like the expand information.

#### URI-related Tasks ####

##### Context URL #####

The context URL is a mandatory part of all data-related responses of an OData
service except for responses with no metadata at all. In some cases it cannot
be determined from the data alone that is passed to the serialization
methods. Therefore, the serialization methods expect the correct context URL
as parameter.

The `ContextURL` object has its own builder that allows to build the context
URL from its parts. For the difficult parts the `UriHelper` instance to be
got from the `OData` object's `createUriHelper()` method has helper methods
that assist with building select lists and key predicates.

##### Canonical URL #####

The `UriHelper` instance to be got from the `OData` object's
`createUriHelper()` method has a helper method to construct the canonical URL
of an entity. This URL can be used as `Location` HTTP header, for example.

##### URI parser #####

The `UriHelper` instance to be got from the `OData` object's
`createUriHelper()` method has a helper method to parse the entity-ID URI of
an entity. This method can be used in the handling of reference-changing
requests.

#### Helpers for Concurrency Control ####

The OData standard defines optimistic concurrency control, a mechanism to
ensure that a modification request accesses the current version of the data
to be modified. Furthermore, this mechanism can be used to enable client-side
caching, using the information that the requested data are still current.

To achieve this, OData uses weak entity tags via the `ETag` HTTP response
header and its associated `If-Match` and `If-None-Match` HTTP request
headers.

To enable this functionality for run-time data, service implementations can
register a class that implements the `CustomETagSupport` interface with its
two methods for entities and media-entity data. These methods should return
for a given entity-set or singleton name whether the service supports entity
tags for entities out of this entity set or singleton. No finer granularity
is possible currently. Processing methods still have to check the ETag(s)
themselves; the `ETagHelper` instance to be got from the `OData` object's
`createETagHelper()` method has two ready-made methods for read and change
requests, respectively.

To enable this functionality for metadata, i.e., for the service document and
the metadata document, service implementations can pass an instance of an
implementation of the `ServiceMetadataETagSupport` interface to the
`ServiceMetadata` creation described above. The default processors for
service- and metadata document requests already take this ETag support into
account.

#### Helpers for Preference Handling ####

Some aspects of OData request processing can be influenced from a client by
setting preferences in the HTTP `Prefer` header. The service implementation
is not obliged to honor these preferences. If it does, it should respond with
an appropriate `Preference-Applied` HTTP header.

The `Preferences` instance to be got from the `OData` object's
`createPreferences()` method with the HTTP `Prefer` headers as parameter
has named access methods for the preferences defined in the OData standard
plus generic access to other preferences.

The `PreferencesApplied` API class has a builder that helps building a
correct `Preference-Applied` HTTP response header.

#### Debug Output ####

For support purposes there is a possibility to enrich the service response
with additional data helpful for finding bugs. Please note that this
information could also help attackers.

To enable this functionality, service implementations can register a class
that implements the `DebugSupport` interface with its two methods for user
authorization and creation of output.

A ready-made `DefaultDebugSupport` class is already provided where all users
are always authorized and the additional data consists of information about
the request, the response, the parsed request URI, the server environment,
library timings, and the stacktrace in case an error occurred, in a
self-contained HTML document ready for browser usage or in a JSON document.
The `OData` object's `createDebugResponseHelper()` method returns a
`DebugResponseHelper` instance which is used in this default implementation.

To request the debug output for a request to the OData service the query parameter
`odata-debug=html` must be appended to the original request URL 
(e.g. `http://localhost:8080/odata-server-tecsvc/odata.svc/?odata-debug=html` for a local published test service).

Definition of the Web Infrastructure
------------------------------------

To enable an OData service on a web server, the service is wrapped by a web
application.

The web application is defined in a `web.xml` file where a servlet is
registered. The servlet is a standard `HttpServlet` configured to dispatch
all requests to URLs below the service's root URL to the OData handler class.
This is done by overriding the servlet's `service()` method. The library
provides an `ODataHttpHandler` object, creatable from the `OData` API class
with the EDM definition as parameter, that can be used inside the `service()`
method.

It receives the request and delegates it to the processor implementation of
the OData service if the URL conforms to the OData specification. This means
that all processor implementations of the OData service have to be registered
with the `register()` method. The responses of the registered handlers are
given back to the servlet infrastructure and will result in corresponding
HTTP responses.
