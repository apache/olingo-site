Title: How to build an OData Service with Olingo V4

# How to build an OData Service with Olingo V4
# Part 7: Media Entities

## Introduction


In the present tutorial, we will implement a media entity set.

**Note:**
The final source code can be found in the project [git repository](https://gitbox.apache.org/repos/asf/olingo-odata4).
A detailed description how to checkout the tutorials can be found [here](/doc/odata4/tutorials/prerequisites/prerequisites.html).
This tutorial can be found in subdirectory /samples/tutorials/p10_media


**Table of Contents**

[TOC]

## Preparation

You should read the previous tutorials first to have an idea how to read entities and entity collections. In addition the following code is based on the write tutorial merged with the navigation tutorial.

As a shortcut you should checkout the prepared tutorial project in the git repository in folder /samples/tutorials/p9_action_preparation.

Afterwards do a Deploy and run: it should be working. At this state you can perform CRUD operations and do navigations between products and categories.

## Implementation

In this tutorial we will implement a media entity set. The idea is to store advertisements and a related image. The metadata document have to be extended by the following elements:

```xml
    <EntityType Name="Advertisement" HasStream="true">
        <Key>
            <PropertyRef Name="ID"/>
        </Key>
        <Property Name="ID" Type="Edm.Guid"/>
        <Property Name="Name" Type="Edm.String"/>
        <Property Name="AirDate" Type="Edm.DateTimeOffset"/>
    </EntityType>

    <EntityContainer Name="Container">
        <EntitySet Name="Advertisements" EntityType="OData.Demo.Advertisement"/>
    </EntityContainer>
```

As you can see, the XML tag `EntityType` has a property `HasStream`, which tells us that this entity has an associated media stream. Such an Entity consists of common properties like *ID* and *Name* and the media stream.

**Tasks**

 * Extend the metadata document
 * Enable the data store to handle media entities
 * Implement the interface `MediaEntityProcessor`

### Extend the metadata document

If you have read the previous tutorials, you should be familiar with the definition entity types. The only difference to regular (Non media enties) is, that they have a `hasStream` property. If this property is not provided it defaults to false. So add the following code to class `DemoEdmProvider`:

We start with method `DemoEdmProvider.getEntityType`

```java
    public CsdlEntityType getEntityType(FullQualifiedName entityTypeName) {
        CsdlEntityType entityType = null;

        if (entityTypeName.equals(ET_PRODUCT_FQN)) {
            // Definition of entity type Product
            // ...
        } else if (entityTypeName.equals(ET_CATEGORY_FQN)) {
            // Definition of entity type Category
            // ...
        } else if(entityTypeName.equals(ET_ADVERTISEMENT_FQN)) {
            CsdlProperty id = new CsdlProperty().setName("ID")
                                    .setType(EdmPrimitiveTypeKind.Guid.getFullQualifiedName());
            CsdlProperty name = new CsdlProperty().setName("Name")
                                    .setType(EdmPrimitiveTypeKind.String.getFullQualifiedName());
            CsdlProperty airDate = new CsdlProperty().setName("AirDate")
                                    .setType(EdmPrimitiveTypeKind.DateTimeOffset.getFullQualifiedName());

            CsdlPropertyRef propertyRef = new CsdlPropertyRef();
            propertyRef.setName("ID");

            entityType = new CsdlEntityType();
            entityType.setName(ET_ADVERTISEMENT_NAME);
            entityType.setProperties(Arrays.asList(id, name, airDate));
            entityType.setKey(Collections.singletonList(propertyRef));
            entityType.setHasStream(true);    // <- Enable the media entity stream
        }

        return entityType;
    }
```

Further we  have to create a new entity set. Add the following snipped to `DemoEdmProvider.getEntitySet`

```java
    @Override
    public CsdlEntitySet getEntitySet(FullQualifiedName entityContainer, String entitySetName) {
        CsdlEntitySet entitySet = null;

        if (entityContainer.equals(CONTAINER)) {
            if (entitySetName.equals(ES_PRODUCTS_NAME)) {
                // Definition of entity set Products
            } else if (entitySetName.equals(ES_CATEGORIES_NAME)) {
                // Definition if entity set Categories
            } else if (entitySetName.equals(ES_ADVERTISEMENTS_NAME)) {
                entitySet = new CsdlEntitySet();
                entitySet.setName(ES_ADVERTISEMENTS_NAME);
                entitySet.setType(ET_ADVERTISEMENT_FQN);
            }
        }

        return entitySet;
    }
```

And finally announce the entity type and entity set:

```java
    @Override
    public List<CsdlSchema> getSchemas() {
        // ...
        entityTypes.add(getEntityType(ET_ADVERTISEMENT_FQN));
        // ...

        return schemas;
    }

    public CsdlEntityContainer getEntityContainer() {
        // ...
        entitySets.add(getEntitySet(CONTAINER, ES_ADVERTISEMENTS_NAME));
    }
```

### Enable the data store to handle media entities

In this tutorial, we will keep things simple. To store the value of media entities, we create a special property *$value*. Note this is not a valid OData Identifier.
All methods have to be implemented in class `myservice.mynamespace.data.Storage`

To read the content to a media entity, we simple return the value of the property *$value*.

```java
    private static final String MEDIA_PROPERTY_NAME = "$value";
    private List<Entity> advertisements;

    public byte[] readMedia(final Entity entity) {
        return (byte[]) entity.getProperty(MEDIA_PROPERTY_NAME).asPrimitive();
    }
```

If we update the content of a media entity, we must also set the the Content Type of the content.

```java
    public void updateMedia(final Entity entity, final String mediaContentType, final byte[] data) {
        entity.getProperties().remove(entity.getProperty(MEDIA_PROPERTY_NAME));
        entity.addProperty(new Property(null, MEDIA_PROPERTY_NAME, ValueType.PRIMITIVE, data));
        entity.setMediaContentType(mediaContentType);
    }
```

If a client creates a new media entity, the body of the requet contains the content of the media entity instead the regular properties! So the other regular properties defaults to `null`. The Content Type of the  media content must also be set.

```java
    public Entity createMediaEntity(final EdmEntityType edmEntityType, final String mediaContentType, final byte[] data) {
        Entity entity = null;

        if(edmEntityType.getName().equals(DemoEdmProvider.ET_ADVERTISEMENT_NAME)) {
            entity = new Entity();
            entity.addProperty(new Property(null, "ID", ValueType.PRIMITIVE, UUID.randomUUID()));
            entity.addProperty(new Property(null, "Name", ValueType.PRIMITIVE, null));
            entity.addProperty(new Property(null, "AirDate", ValueType.PRIMITIVE, null));

            entity.setMediaContentType(mediaContentType);
            entity.addProperty(new Property(null, MEDIA_PROPERTY_NAME, ValueType.PRIMITIVE, data));

            advertisements.add(entity);
        }

        return entity;
    }
```

Add an initial set of data to our data store:

```java
    private void initAdvertisementSampleData() {
        Entity entity = new Entity();
        entity.addProperty(new Property(null, "ID", ValueType.PRIMITIVE, UUID.fromString("f89dee73-af9f-4cd4-b330-db93c25ff3c7")));
        entity.addProperty(new Property(null, "Name", ValueType.PRIMITIVE, "Old School Lemonade Store, Retro Style"));
        entity.addProperty(new Property(null, "AirDate", ValueType.PRIMITIVE, Timestamp.valueOf("2012-11-07 00:00:00")));
        entity.addProperty(new Property(null, MEDIA_PROPERTY_NAME, ValueType.PRIMITIVE, "Super content".getBytes()));
        entity.setMediaContentType(ContentType.parse("text/plain").toContentTypeString());
        advertisements.add(entity);

        entity = new Entity();
        entity.addProperty(new Property(null, "ID", ValueType.PRIMITIVE,
        UUID.fromString("db2d2186-1c29-4d1e-88ef-a127f521b9c67")));
        entity.addProperty(new Property(null, "Name", ValueType.PRIMITIVE, "Early morning start, need coffee"));
        entity.addProperty(new Property(null, "AirDate", ValueType.PRIMITIVE, Timestamp.valueOf("2000-02-29 00:00:00")));
        entity.addProperty(new Property(null, MEDIA_PROPERTY_NAME, ValueType.PRIMITIVE, "Super content2".getBytes()));
        entity.setMediaContentType(ContentType.parse("text/plain").toContentTypeString());
        advertisements.add(entity);
    }
```

Call `initAdvertisementSampleData()` in the constructor.

```java
    public Storage() {
        // ...
        advertisements = new ArrayList<Entity>();
        // ...
        initAdvertisementSampleData();
    }
```

Enable the regular entity set for CRUD opertations:

```java
    public EntityCollection readEntitySetData(EdmEntitySet edmEntitySet) throws ODataApplicationException {

        if (edmEntitySet.getName().equals(DemoEdmProvider.ES_PRODUCTS_NAME)) {
            // ...
        } else if(edmEntitySet.getName().equals(DemoEdmProvider.ES_ADVERTISEMENTS_NAME)) {
            return getEntityCollection(advertisements);
        }

        return null;
    }

    public Entity readEntityData(EdmEntitySet edmEntitySet, List<UriParameter> keyParams)
            throws ODataApplicationException {

        EdmEntityType edmEntityType = edmEntitySet.getEntityType();
        if (edmEntitySet.getName().equals(DemoEdmProvider.ES_PRODUCTS_NAME)) {
            // ...
        } else if(edmEntitySet.getName().equals(DemoEdmProvider.ES_ADVERTISEMENTS_NAME)) {
            return getEntity(edmEntityType, keyParams, advertisements);
        }

        return null;
    }

    public Entity createEntityData(EdmEntitySet edmEntitySet, Entity entityToCreate) {

        EdmEntityType edmEntityType = edmEntitySet.getEntityType();
        if (edmEntitySet.getName().equals(DemoEdmProvider.ES_PRODUCTS_NAME)) {
            // ....
        } else if(edmEntitySet.getName().equals(DemoEdmProvider.ES_CATEGORIES_NAME)) {
            return createEntity(edmEntityType, entityToCreate, categoryList);
        }

        return null;
    }

    public void updateEntityData(EdmEntitySet edmEntitySet, List<UriParameter> keyParams,
            Entity updateEntity, HttpMethod httpMethod) throws ODataApplicationException {

        EdmEntityType edmEntityType = edmEntitySet.getEntityType();
        if (edmEntitySet.getName().equals(DemoEdmProvider.ES_PRODUCTS_NAME)) {
            // ...
        } else if(edmEntitySet.getName().equals(DemoEdmProvider.ES_ADVERTISEMENTS_NAME)) {
            updateEntity(edmEntityType, keyParams, updateEntity, httpMethod, advertisements);
        }
    }

    public void deleteEntityData(EdmEntitySet edmEntitySet, List<UriParameter> keyParams)
            throws ODataApplicationException {

        EdmEntityType edmEntityType = edmEntitySet.getEntityType();
        if (edmEntitySet.getName().equals(DemoEdmProvider.ES_PRODUCTS_NAME)) {
            // ...
        } else if(edmEntitySet.getName().equals(DemoEdmProvider.ES_ADVERTISEMENTS_NAME)) {
            deleteEntity(edmEntityType, keyParams, advertisements);
        }
    }
```

### Implement the interface `MediaEntityProcessor`

As you can see the [`MediaEntityProcessor`(Javadoc)](/javadoc/odata4/org/apache/olingo/server/api/processor/MediaEntityProcessor.html) extends [`EntityProcessor`](/javadoc/odata4/org/apache/olingo/server/api/processor/EntityProcessor.html), therefore we will implement `MediaEntityProcessor` in class `DemoEntityProcessor`.

The easiest part is to delete an media entity. The method `deleteMediaEntity` is delegated to the method `deleteEntity(...)`.

```java
    @Override
    public void deleteMediaEntity(ODataRequest request, ODataResponse response, UriInfo uriInfo)
            throws ODataApplicationException, ODataLibraryException {

        /*
         * In this tutorial, the content of the media entity is stored in a special property.
         * So no additional steps to delete the content of the media entity are necessary.
         *
         * A real service may store the content on the file system. So we have to take care to
         * delete external files too.
         *
         * DELETE request to /Advertisements(ID) will be dispatched to the deleteEntity(...) method
         * DELETE request to /Advertisements(ID)/$value will be dispatched to the deleteMediaEntity(...) method
         *
         * So it is a good idea handle deletes in a central place.
         */

        deleteEntity(request, response, uriInfo);
    }
```

Next the creation of media entites is implemented. First we fetch the addressed entity set and convert the body of the request to a byte array. Remember the whole body of the request contains the content of the media entity.

```java
    @Override
    public void createMediaEntity(ODataRequest request, ODataResponse response, UriInfo uriInfo,
            ContentType requestFormat, ContentType responseFormat)
            throws ODataApplicationException, ODataLibraryException {

        final EdmEntitySet edmEntitySet = Util.getEdmEntitySet(uriInfo);
        final byte[] mediaContent = odata.createFixedFormatDeserializer().binary(request.getBody());
        ///...
```

After that we call the data store to create the new media entity. The [OData Specification](http://docs.oasis-open.org/odata/odata/v4.0/errata02/os/complete/part1-protocol/odata-v4.0-errata02-os-part1-protocol-complete.html#_Toc406398337) tells us, that we have to set the location header to the edit URL of the entity. Since we do not support Prefer Headers we have to return the entity itself.

```java
        final Entity entity = storage.createMediaEntity(edmEntitySet.getEntityType(),
        requestFormat.toContentTypeString(),mediaContent);

        final ContextURL contextUrl = ContextURL.with().entitySet(edmEntitySet).suffix(Suffix.ENTITY).build();
        final EntitySerializerOptions opts = EntitySerializerOptions.with().contextURL(contextUrl).build();
        final SerializerResult serializerResult = odata.createSerializer(responseFormat)
                            .entity(serviceMetadata, edmEntitySet.getEntityType(), entity, opts);

        final String location = request.getRawBaseUri() + '/'
                + odata.createUriHelper().buildCanonicalURL(edmEntitySet, entity);

        response.setContent(serializerResult.getContent());
        response.setStatusCode(HttpStatusCode.CREATED.getStatusCode());
        response.setHeader(HttpHeader.LOCATION, location);
        response.setHeader(HttpHeader.CONTENT_TYPE, responseFormat.toContentTypeString());
    }
```

To keep things simple, our scenario do not support navigation to media entities. Because of this, the implementation to read a media entity is quite simple. First analayse the URI and fetch the entity. Than take the content of our specical property, serialize them and return the serialized content. The serializer converts the byte array to an `InputStream`.

```java
    @Override
    public void readMediaEntity(ODataRequest request, ODataResponse response, UriInfo uriInfo, ContentType responseFormat)
            throws ODataApplicationException, ODataLibraryException {

        // Since our scenario do not contain navigations from media entities. We can keep things simple and
        // check only the first resource path of the URI.
        final UriResource firstResoucePart = uriInfo.getUriResourceParts().get(0);
        if(firstResoucePart instanceof UriResourceEntitySet) {
            final EdmEntitySet edmEntitySet = Util.getEdmEntitySet(uriInfo);
            final UriResourceEntitySet uriResourceEntitySet = (UriResourceEntitySet) firstResoucePart;

            final Entity entity = storage.readEntityData(edmEntitySet, uriResourceEntitySet.getKeyPredicates());
            if(entity == null) {
                throw new ODataApplicationException("Entity not found",
                    HttpStatusCode.NOT_FOUND.getStatusCode(), Locale.ENGLISH);
            }

            final byte[] mediaContent = storage.readMedia(entity);
            final InputStream responseContent = odata.createFixedFormatSerializer().binary(mediaContent);

            response.setStatusCode(HttpStatusCode.OK.getStatusCode());
            response.setContent(responseContent);
            response.setHeader(HttpHeader.CONTENT_TYPE, entity.getMediaContentType());
        } else {
            throw new ODataApplicationException("Not implemented",
                HttpStatusCode.BAD_REQUEST.getStatusCode(), Locale.ENGLISH);
        }
    }
```

Updating a media entity in our scenario is quite similar to read an entity. The first step is to analyse the URI, than fetch the entity from data store. Afer that we call the `updateMediaEntity` method. In our case we do not return any content. If we would return content, we must return the recently uploaded content of the media entity ([OData Version 4.0 Part 1: Protocol](http://docs.oasis-open.org/odata/odata/v4.0/errata02/os/complete/part1-protocol/odata-v4.0-errata02-os-part1-protocol-complete.html#_Toc406398338)).

```java
    @Override
    public void updateMediaEntity(ODataRequest request, ODataResponse response, UriInfo uriInfo,
            ContentType requestFormat, ContentType responseFormat)
            throws ODataApplicationException, ODataLibraryException {

        final UriResource firstResoucePart = uriInfo.getUriResourceParts().get(0);
        if (firstResoucePart instanceof UriResourceEntitySet) {
            final EdmEntitySet edmEntitySet = Util.getEdmEntitySet(uriInfo);
            final UriResourceEntitySet uriResourceEntitySet = (UriResourceEntitySet) firstResoucePart;

            final Entity entity = storage.readEntityData(edmEntitySet, uriResourceEntitySet.getKeyPredicates());
            if (entity == null) {
                throw new ODataApplicationException("Entity not found",
                    HttpStatusCode.NOT_FOUND.getStatusCode(), Locale.ENGLISH);
            }

            final byte[] mediaContent = odata.createFixedFormatDeserializer().binary(request.getBody());
            storage.updateMedia(entity, requestFormat.toContentTypeString(), mediaContent);

            response.setStatusCode(HttpStatusCode.NO_CONTENT.getStatusCode());
        } else {
            throw new ODataApplicationException("Not implemented",
                HttpStatusCode.NOT_IMPLEMENTED.getStatusCode(), Locale.ENGLISH);
        }
    }
```

## Run the implemented service

After building and deploying the project, we can invoke our OData service.

 * Read media entity set
 **GET** [http://localhost:8080/DemoService-Media/DemoService.svc/Advertisements](http://localhost:8080/DemoService-Media/DemoService.svc/Advertisments)

 * Read media entity
 **GET** [http://localhost:8080/DemoService-Media/DemoService.svc/Advertisements(f89dee73-af9f-4cd4-b330-db93c25ff3c7)](http://localhost:8080/DemoService-Media/DemoService.svc/Advertisments(f89dee73-af9f-4cd4-b330-db93c25ff3c7))

 * Read media entity content
 **GET** [http://localhost:8080/DemoService-Media/DemoService.svc/Advertisements(f89dee73-af9f-4cd4-b330-db93c25ff3c7)/$value](http://localhost:8080/DemoService-Media/DemoService.svc/Advertisments(f89dee73-af9f-  4cd4-b330-db93c25ff3c7)/$value)

 * Create a new Media Entity
 **POST** [http://localhost:8080/DemoService-Media/DemoService.svc/Advertisements

Content-Type: image/svg+xml



```xml
    <?xml version="1.0" encoding="UTF-8"?>
        <svg xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 100 100">
            <g stroke="darkmagenta" stroke-width="16" fill="crimson">
                <circle cx="50" cy="50" r="42"/>
            </g>
    	</svg>
```

 * Update the content of a media entity
 **PUT**  [http://localhost:8080/DemoService-Media/DemoService.svc/Advertisements(f89dee73-af9f-4cd4-b330-db93c25ff3c7)/$value](http://localhost:8080/DemoService-Media/DemoService.svc/Advertisments(f89dee73-af9f-4cd4-b330-db93c25ff3c7)/$value)

Content-Type: text/plain


    Super super nice content

 * Update the properties of a media entity
**PUT** [http://localhost:8080/DemoService-Media/DemoService.svc/Advertisements(f89dee73-af9f-4cd4-b330-db93c25ff3c7)](http://localhost:8080/DemoService-Media/DemoService.svc/Advertisments(f89dee73-af9f-4cd4-b330-db93c25ff3c7))

Content-Type: application/json


```json
    {
        "Name": "New Name",
        "AirDate": "2020-06-05T23:00"
    }
```


 * Delete a media entity
 **DELETE** [http://localhost:8080/DemoService-Media/DemoService.svc/Advertisements(f89dee73-af9f-4cd4-b330-db93c25ff3c7)](http://localhost:8080/DemoService-Media/DemoService.svc/Advertisments(f89dee73-af9f-4cd4-b330-db93c25ff3c7))

 * Delete a media entity
 **DELETE** [http://localhost:8080/DemoService-Media/DemoService.svc/Advertisements(db2d2186-1c29-4d1e-88ef-127f521b9c67)/$value](http://localhost:8080/DemoService-Media/DemoService.svc/Advertisments(db2d2186-1c29-4d1e-88ef-127f521b9c67)/$value)

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
  * Tutorial OData V4 service, part 7: Media Entities
  * Tutorial OData V4 service, part 8: [Batch Request support](/doc/odata4/tutorials/batch/tutorial_batch.html)
  * Tutorial OData V4 service, part 9: [Handling "Deep Insert" requests](/doc/odata4/tutorials/deep_insert/tutorial_deep_insert.html)
  
### Code and Repository
  * [Git Repository](https://gitbox.apache.org/repos/asf/olingo-odata4)
  * [Guide - To fetch the tutorial sources](/doc/odata4/tutorials/prerequisites/prerequisites.html)
  * [Demo Service source code as zip file (contains all tutorials)](http://www.apache.org/dyn/closer.lua/olingo/odata4/4.0.0/DemoService_Tutorial.zip)

### Further reading

  * [Official OData Homepage](http://odata.org/)
  * [OData documentation](http://www.odata.org/documentation/)
  * [Olingo Javadoc](/javadoc/odata4/index.html)
