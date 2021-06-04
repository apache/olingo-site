Title: How to build an OData Service with Olingo V4

# How to build an OData Service with Olingo V4

# Add Streaming Support (for Entity Collections)
Available with *Apache Olingo 4.2.0 (and newer).*

## Preface

In the present tutorial we will add streaming support for Entity Collections on a per Entity granularity.

**Note:**
The final source code can be found in the project [git repository](https://gitbox.apache.org/repos/asf/olingo-odata4).  
A detailed description how to checkout the tutorials can be found [here](/doc/odata4/tutorials/prerequisites/prerequisites.html).  
This tutorial can be found in the `DemoService-Streaming` module in the projects subdirectory `/samples/tutorials/pe_streaming`

**Table of Contents**

  1. Introduction
  1. Preparation
  1. Implementation
  1. Run the implemented service
  1. Links

# Introduction
The actual *streaming support* in the Olingo library enables a way to provide an Entity Collection on a single Entity granularity. This enables support for e.g. *chunked HTTP responses* without the need to have the whole Entity Collection pre-loaded (and probably in memory). Therefore the `EntityIterator` interface is used to check for additional entities and to provide the next available entity. The how a single Entity is provided is than completely based on the decision of the service developer.

A possible implementation then could e.g. pre-load ten entities and serve them as *chunked HTTP responses* and first with the next requested chunkes the next (ten) entities would be loaded from the database. With such an implementation the runtime memory consumption could be reduced (with the counterpart of more database round trips) and the client has the possibility to visualise the already delivered entities (if the client support this).

# Preparation
You should read the previous tutorials first to have an idea how to read entity collections. In addition the following code is based on the [read collection tutorial (Part 2)][re_co_tu].

As a shortcut for the upcoming modification steps you should checkout the mentioned tutorial project. It is available in the git repository in folder `/samples/tutorials/p2_readep` (for more information about checkout see in the [read collection tutorial (Part 2)][re_co_tu]).

The main idea of the following implementation is to enable a basic streaming support in the sample *data provider* (the `Storage` class) and use this in the already existing processors.

Therefore following steps have to be performed:

  * Modify the *data provider* (the `Storage` class)
    * Includes a basic implementation of the `EntityIterator` interface
  * Use the `EntityIterator` in the `readEntityCollection(..)` method
  * Optional: Add exception/error handling (with `ODataContentWriteErrorCallback`)

# Implementation
To enable the *streaming support* in a service there are following steps which need to be done:

  1. An `EntityIterator` implementation has to be used to provide the entity collection data (`Entity` objects)
  1. This `EntityIterator` has to be passed to the `entityCollectionStreamed(...)` method of the used `ODataSerializer`
  1. The `ODataSerializer` than returns a `SerializerStreamResult` which contains the *stream enabled* result within a `ODataContent` object.
  1. The `ODataContent` is then set at the `ODataResponse` via the `setODataContent(...)` method

Basically it is the same as in the *none streaming* with the difference that some other objects and classes has to be used.

For demonstration of above steps the existing [read collection tutorial][re_co_tu] will be now enabled for *streaming of entity collections*.

## Simplest approach
The *simplest approach* is to wrap the already existing `EntityCollection` into an `EntityIterator` and pass this to the according `entityCollectionStreamed(...)` method.
With this the service would not change how the data is accessed but would (easily) enable the possibility for a (streamed) *chunked HTTP response* (if this is supported by the environment e.g. JEE application server).

In the existing [read collection tutorial][re_co_tu] following new method is necessary to create an `EntityIterator` to wrap an `EntityCollection`:

~~~java
    private EntityIterator wrapAsIterator(final EntityCollection collection) {
      final Iterator<Entity> it = collection.iterator();
      return new EntityIterator() {
        @Override
        public boolean hasNext() {
          return it.hasNext();
        }

        @Override
        public Entity next() {
          return it.next();
        }
      };
    }
~~~

The (as anonymous inner class) created `EntityIterator` only iterates over the already loaded entities (of the `EntityCollection`).

To use this `EntityIterator` in the `readEntityCollection(..)` method the `EntityIterator` must be passed to the `ODataSerializer` via the `entityCollectionStreamed(...)` method and the `ODataContent` object of the resulting `SerializerStreamResult` must be set at the `ODataResponse` via the `setODataContent(...)` method.
What sound like a lot to do is just the below code snippet:

~~~java
      ...
      EntityIterator iterator = wrapAsIterator(entityCollection);
      SerializerStreamResult serializerResult = serializer.entityCollectionStreamed(serviceMetadata,
          edmEntityType, iterator, opts);

      // 4th: configure the response object: set the body, headers and status code
      response.setODataContent(serializerResult.getODataContent());
      ...
    }
~~~

Which replaces following original code snippet:

~~~java
      ...
      SerializerResult serializerResult = serializer.entityCollection(serviceMetadata,
          edmEntityType, entityCollection, opts);

      // 4th: configure the response object: set the body, headers and status code
      response.setContent(serializedContent);
      ...
    }
~~~


## DataProvider based approach
The *realistic approach* is that the data provider (e.g. a database) creates an `EntityIterator` which is used to provide the entity collection data (`Entity` objects) to the `EntityProcessor` and `ODataSerializer`.

With this approach not only the option for a (streamed) chunked HTTP response (if this is supported by the environment e.g. JEE application server) is enabled. Furthermore the data provider is in charge at which time how many entities are loaded (and hold) in memory.
This means as example, that a data provider can implement a concept of lazy loading of the entity collection in which e.g. a database connection is established but only the first ten entities are loaded in memory and passed for response serialization. First when the serializer need the eleventh (and/or more) entity those are loaded from the database (and the first ten can be removed from memory).
Practically such an approach requires more database roundtrips but also a smaller memory footprint and less eager loading at the begin of the request/response cycle.

In the existing [read collection tutorial][re_co_tu] the `Storage` class is used a data provider (acting like a database).
For enablement of the *streaming support* following new method is introduced which create an `EntityIterator` to allow the iterable passed access to the stored entities:

~~~java
    public EntityIterator readEntitySetDataStreamed(EdmEntitySet edmEntitySet)throws ODataApplicationException {
      // actually, this is only required if we have more than one Entity Sets
      if(edmEntitySet.getName().equals(DemoEdmProvider.ES_PRODUCTS_NAME)){
        final Iterator<Entity> it = productList.iterator();
        return new EntityIterator() {
          @Override
          public boolean hasNext() {
            return it.hasNext();
          }

          @Override
          public Entity next() {
            return it.next();
          }
        };
      }

      return null;
    }
~~~

As described above in the existing implementation the use of the `EntityCollection` has to be replaced with the `EntityIterator`, which means that this line:
`EntityCollection entityCollection = storage.readEntitySetData(edmEntitySet);`
has to be replaced by that line:
`EntityIterator iterator = storage.readEntitySetDataStreamed(edmEntitySet);`

And the

~~~java
    SerializerResult serializerResult = serializer.entityCollection(
      serviceMetadata, edmEntityType, entityCollection, opts);
~~~

has to be replaced by

~~~java
    SerializerStreamResult serializerResult = serializer.entityCollectionStreamed(
      serviceMetadata, edmEntityType, iterator, opts);
~~~

And at the `ODataResponse` now instead of:
`response.setContent(serializerResult.getContent());`
the result is set as `ODataContent`:
`response.setODataContent(serializerResult.getODataContent());`

As result the whole `readEntityCollection(...)` method now look like following:

~~~java
    public void readEntityCollection(ODataRequest request, ODataResponse response, UriInfo uriInfo, ContentType responseFormat) throws ODataApplicationException, SerializerException {

      // 1st retrieve the requested EntitySet from the uriInfo (representation of the parsed URI)
      List<UriResource> resourcePaths = uriInfo.getUriResourceParts();
      UriResourceEntitySet uriResourceEntitySet = (UriResourceEntitySet) resourcePaths.get(0); // in our example, the first segment is the EntitySet
      EdmEntitySet edmEntitySet = uriResourceEntitySet.getEntitySet();

      // 2nd: fetch the data from backend for this requested EntitySetName and deliver as EntitySet
      EntityIterator iterator = storage.readEntitySetDataStreamed(edmEntitySet);

      // 3rd: create a serializer based on the requested format (json)
      ODataSerializer serializer = odata.createSerializer(responseFormat);

      // and serialize the content: transform from the EntitySet object to InputStream
      EdmEntityType edmEntityType = edmEntitySet.getEntityType();
      ContextURL contextUrl = ContextURL.with().entitySet(edmEntitySet).build();

      final String id = request.getRawBaseUri() + "/" + edmEntitySet.getName();
      EntityCollectionSerializerOptions opts = EntityCollectionSerializerOptions.with().id(id)
              .contextURL(contextUrl).build();

      SerializerStreamResult serializerResult = serializer.entityCollectionStreamed(serviceMetadata,
          edmEntityType, iterator, opts);

      // 4th: configure the response object: set the body, headers and status code
      response.setODataContent(serializerResult.getODataContent());
      response.setStatusCode(HttpStatusCode.OK.getStatusCode());
      response.setHeader(HttpHeader.CONTENT_TYPE, responseFormat.toContentTypeString());
    }
~~~

After this changes the data access (encapsulated in the `EntityIterator`) and serialization is now done directly when the data is processed by the web framework layer (e.g. JEE servlet layer) and not within the call hierarchy of the `readEntityCollection(...)` method.

The *counterpart* of this is that when an error/exception occurs during the serialization of the data the `readEntityCollection(...)` method already returned and hence there is no possibility (at this point) to catch the exception and do an error handling.
Furthermore because of the *streaming* the *HTTP Header* is already sent to the client (with e.g. a `HTTP Status-Code: 200 OK`).
Based on this the *OData-v4.0 Part1 Protocol* describes in chapter *9.4 In-Stream Errors* how to handle this:
> In the case that the service encounters an error after sending a success status to the client, the service MUST generate an error within the payload, which may leave the response malformed. Clients MUST treat the entire response as being in error. This specification does not prescribe a particular format for generating errors within a payload.

And for Olingo exists the `ODataContentWriteErrorCallback` which is described in the chapter *Exception/Error Handling*.

### More realistic data provider

Because the simplistic data provider in the tutorial the `EntityIterator` is also very simplistic.
However it is also realistic to have an `EntityIterator` which e.g. access a database result set which is `next():Entity` call (see below code snippet to get the idea).

~~~java
    public class MyEntityIterator extends EntityIterator {
      ResultSet set; //...

      public MyEntityIterator(ResultSet set) {
        this.set = set;
      }

      @Override
      public boolean hasNext() {
        return set.next();
      }

      @Override
      public Entity next() {
        return readNextEntityFromResultSet();
      }

      private Entity readNextEntityFromResultSet() {
        // read data from result set and return as entity object
      }
    }
~~~

## Exception/Error Handling
The *counterpart* of the *streaming support* is that when an error/exception occurs during the serialization of the data the service implementation is not in charge anymore to catch the exception and do an error handling.

Furthermore because of the *streaming* the *HTTP Header* is already sent to the client (with e.g. a `HTTP Status-Code: 200 OK`).
Based on this the *OData-v4.0 Part1 Protocol* describes in chapter *9.4 In-Stream Errors* how to handle this:
> In the case that the service encounters an error after sending a success status to the client, the service MUST generate an error within the payload, which may leave the response malformed. Clients MUST treat the entire response as being in error. This specification does not prescribe a particular format for generating errors within a payload.

For *exception/error handling* in Olingo exists the `ODataContentWriteErrorCallback` interface which must be implemented and then can be set as an option at the `EntityCollectionSerializerOptions` with the `writeContentErrorCallback(...)` method.

If during processing (*write*) of the `ODataContent` object (normally serialization into an according `OutputStream`, like the `javax.servlet.ServletOutputStream` in a JEE servlet environment) an exception occurs the `ODataContentWriteErrorCallback` `handleError` method is called.
This method get as parameter the `ODataContentWriteErrorContext` which contains at least the thrown and to be handled `Exception` and the `WritableByteChannel` in which the payload of the response was written before the error occurred.
Based on the requirements of the OData specification that *the service MUST generate an error within the payload, which may leave the response malformed* the `WritableByteChannel` is still open and the service developer can write additional data to ensure that the response payload is malformed.

A basic `ODataContentWriteErrorCallback` implementation could look like this code snippet:

~~~java
    private ODataContentWriteErrorCallback errorCallback = new ODataContentWriteErrorCallback() {
      public void handleError(ODataContentWriteErrorContext context, WritableByteChannel channel) {
        String message = "An error occurred with message: ";
        if(context.getException() != null) {
          message += context.getException().getMessage();
        }
        try {
          channel.write(ByteBuffer.wrap(message.getBytes()));
        } catch (IOException e) {
          throw new RuntimeException(e);
        }
      }
    };
~~~

And could be set in the [read collection tutorial][re_co_tu] at the `EntityCollectionSerializerOptions` via the `.writeContentErrorCallback(errorCallback)` method.

~~~java
    EntityCollectionSerializerOptions opts =
        EntityCollectionSerializerOptions.with().id(id)
            .writeContentErrorCallback(errorCallback)
            .contextURL(contextUrl).build();
~~~

# Run sample service
After building and deploying your service to your server, you can try a requests to the entity set via: [http://localhost:8080/DemoService/DemoService.svc/Products?$format=json](http://localhost:8080/DemoService/DemoService.svc/Products?$format=json)

The response is exactly the same response as in the none streaming request. So unfortunaly here is no difference beside of the technical fact that the response is serialized at the very end of the request chain and directly written into the response output stream (`javax.servlet.ServletOutputStream`)

~~~json
    {
      "@odata.context": "$metadata#Products",
      "value": [
        {
          "ID": 1,
          "Name": "Notebook Basic 15",
          "Description": "Notebook Basic, 1.7GHz - 15 XGA - 1024MB DDR2 SDRAM - 40GB"
        },
        {
          "ID": 2,
          "Name": "1UMTS PDA",
          "Description": "Ultrafast 3G UMTS/HSDPA Pocket PC, supports GSM network"
        },
        {
          "ID": 3,
          "Name": "Ergo Screen",
          "Description": "19 Optimum Resolution 1024 x 768 @ 85Hz, resolution 1280 x 960"
        }
      ]
    }
~~~

# Links

### Tutorials

Further topics to be covered by follow-up tutorials:

  * Tutorial OData V4 service part 1: [Read Entity Collection](/doc/odata4/tutorials/read/tutorial_read.html)
  * Tutorial OData V4 service part 2: [Read Entity, Read Property](/doc/odata4/tutorials/readep/tutorial_readep.html)
  * Tutorial OData V4 service part 3: [Write (Create, Update, Delete Entity)](/doc/odata4/tutorials/write/tutorial_write.html)
  * Tutorial OData V4 service, part 4: [Navigation](/doc/odata4/tutorials/navigation/tutorial_navigation.html)
  * Tutorial OData V4 service, part 5.1: [System Query Options $top, $skip, $count (this page)](/doc/odata4/tutorials/sqo_tcs/tutorial_sqo_tcs.html)
  * Tutorial OData V4 service, part 5.2: [System Query Options $select, $expand](/doc/odata4/tutorials/sqo_es/tutorial_sqo_es.html)
  * Tutorial OData V4 service, part 5.3: [System Query Options $orderby](/doc/odata4/tutorials/sqo_o/tutorial_sqo_o.html)
  * Tutorial OData V4 service, part 5.4: [System Query Options $filter](/doc/odata4/tutorials/sqo_f/tutorial_sqo_f.html)
  * Tutorial OData V4 service, part 6: [Action and Function Imports](/doc/odata4/tutorials/action/tutorial_action.html)
  * Tutorial OData V4 service, part 7: [Add Media entities to the service](/doc/odata4/tutorials/media/tutorial_media.html)
  * Tutorial OData V4 service, part 8: [Batch request support](/doc/odata4/tutorials/batch/tutorial_batch.html)
  * Tutorial OData V4 service, part 9: [Handling "Deep Insert" requests](/doc/odata4/tutorials/deep_insert/tutorial_deep_insert.html)

### Code and Repository
  * [Git Repository](https://gitbox.apache.org/repos/asf/olingo-odata4)
  * [Guide - To fetch the tutorial sources](/doc/odata4/tutorials/prerequisites/prerequisites.html)
  * [Demo Service source code as zip file (contains all tutorials)](http://www.apache.org/dyn/closer.lua/olingo/odata4/4.0.0/DemoService_Tutorial.zip)

### Further reading

  * [Official OData Homepage](http://odata.org/)
  * [OData documentation](http://www.odata.org/documentation/)
  * [Olingo Javadoc](/javadoc/odata4/index.html)


[re_co_tu]:</doc/odata4/tutorials/readep/tutorial_readep.html>
