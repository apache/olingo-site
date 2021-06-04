Title:    Tutorial - Write service with Olingo V4

# How to build an OData Service with Olingo V4

# Part 3: Write operations

## Introduction

This tutorial guides you through the steps required to write an OData Service based on the Olingo OData 4.0 Library for Java (based on current release which can be got via the [Download-Page](/doc/odata4/download.html)).

In the first two tutorials ([Read Collection](/doc/odata4/tutorials/read/tutorial_read.html) and [Read Entity](/doc/odata4/tutorials/readep/tutorial_readep.html)), we’ve learned how to build a simple OData service that supports read operations for collection, single entity and property.

In the present tutorial, will cover the write operations, which means creating an entity, modifying an existing entity and deleting an existing entity.


**Note**
The final source code can be found in the project [git repository](https://gitbox.apache.org/repos/asf/olingo-odata4).
A detailed description how to checkout the tutorials can be found [here](/doc/odata4/tutorials/prerequisites/prerequisites.html).
This tutorial can be found in subdirectory *\samples\tutorials\p3_write*


**Disclaimer**
Again, in the present tutorial, will focus only on the relevant implementation, in order to keep the code small and simple.
The sample code shouldn't be reused for advanced scenarios.


**Table of Contents**

  1. Prerequisites
  1. Preparation
  1. Implementation of Read Single Entity
    1. Implement the `createEntity(...)` method
    1. Implement the `updateEntity(...)` method
    1. Implement the `deleteEntity(...)` method
  1. Run the implemented service
    1. Example for **CREATE**
    1. Example for **UPDATE (PUT)**
    1. Example for **UPDATE (PATCH)**
    1. Example for **DELETE**
  1. Summary
  1. Links

---

# 1. Prerequisites

Same prerequisites as in [Tutorial Part 1: Read Entity Collection](/doc/odata4/tutorials/read/tutorial_read.html) and [Tutorial Part 2: Read Entity](/doc/odata4/tutorials/readep/tutorial_readep.html) as well as basic knowledge about the concepts presented in both tutorials.

---

# 2. Preparation

Follow [Tutorial Part 1: Read Entity Collection](/doc/odata4/tutorials/read/tutorial_read.html) and [Tutorial Part 2: Read Entity](/doc/odata4/tutorials/readep/tutorial_readep.html) or as shortcut import the project attached to *Tutorial Part 2* into your Eclipse workspace.

Afterwards do a *Deploy and run*: it should be working.

---

# 3. Implementation

In our sample scenario, we want to create a product, to be added to the list of available products that we maintain in our database-mock.
This product that we want to create will have a name and a description that the user of our service will specify in his HTTP request.
The Olingo library takes this user request, serializes the request body and invokes the corresponding method of our processor class.

In the previous tutorial 2, we’ve already implemented the `EntityProcessor` interface and registered our class in the servlet, but we have not written the implementation for the callback methods that are responsible for the write operations.
This is what we are going to do in the below sections.

## 3.1. Implement the createEntity(...) method

Open the class `myservice.mynamespace.service.DemoEntityProcessor`
Go to the method `createEntity(...)`
The method body should be empty, otherwise delete any content.

**Now, how to implement the method?**
Basically, we have to do the same that we did in the `readEntity(...)` method, but the other way ‘round.
In the `createEntity(...)` method, we have to retrieve the payload from the request and then write it to our mock-database.
Furthermore, we have to return the created entity in the response payload.

Again, we can divide our work into 4 steps:

  1. Analyze the URI
  1. Handle data in backend
  1. Serialize
  1. Configure the response


**In detail**

We have to keep in mind that -for creation - the URL that is executed in our example is the following:

    http://localhost:8080/DemoService/DemoService.svc/Products

It is executed as POST request and contains a request body which looks as follows:

```json
    {
      "ID":4,
      "Name":"Gamer Mouse",
      "Description":"optical mouse - gamer edition"
    }
```

**Steps**

  1. In the implementation, we have to first retrieve the `EntityCollection` and `EntityType` metadata from the `UriInfo` object.
  1. The next step is to create the data in our backend.
  For this purpose, we have to retrieve the data from the HTTP request payload.
  We get the payload from the `ODataRequest` instance as `InputStream`, which can then be deserialized.
  Our `Storage` class is responsible for creating the new product in the backend.
  And for returning the newly created instance.
  The reason is that our OData service has to return the newly created entity in the response body.
  1. From now on the procedure is the same like in the `readEntity(...)` method
  1. The only difference is the status code, that has to be set to **201 - created** in case of success

Please find below the sample code for the *createEntity()* method

```java
public void createEntity(ODataRequest request, ODataResponse response, UriInfo uriInfo,
     ContentType requestFormat, ContentType responseFormat)
    throws ODataApplicationException, DeserializerException, SerializerException {

  // 1. Retrieve the entity type from the URI
  EdmEntitySet edmEntitySet = Util.getEdmEntitySet(uriInfo);
  EdmEntityType edmEntityType = edmEntitySet.getEntityType();

  // 2. create the data in backend
  // 2.1. retrieve the payload from the POST request for the entity to create and deserialize it
  InputStream requestInputStream = request.getBody();
  ODataDeserializer deserializer = this.odata.createDeserializer(requestFormat);
  DeserializerResult result = deserializer.entity(requestInputStream, edmEntityType);
  Entity requestEntity = result.getEntity();
  // 2.2 do the creation in backend, which returns the newly created entity
  Entity createdEntity = storage.createEntityData(edmEntitySet, requestEntity);

  // 3. serialize the response (we have to return the created entity)
  ContextURL contextUrl = ContextURL.with().entitySet(edmEntitySet).build();
  // expand and select currently not supported
  EntitySerializerOptions options = EntitySerializerOptions.with().contextURL(contextUrl).build();

  ODataSerializer serializer = this.odata.createSerializer(responseFormat);
  SerializerResult serializedResponse = serializer.entity(serviceMetadata, edmEntityType, createdEntity, options);

  //4. configure the response object
  response.setContent(serializedResponse.getContent());
  response.setStatusCode(HttpStatusCode.CREATED.getStatusCode());
  response.setHeader(HttpHeader.CONTENT_TYPE, responseFormat.toContentTypeString());
}
```



## 3.2. Implement the updateEntity(...) method

Example URL

    http://localhost:8080/DemoService/DemoService.svc/Products(3)

Example request body:

```json
    {
      "ID":3,
      "Name":"Ergo Screen updated Name",
      "Description":"updated description"
    }
```

The `updateEntity(...)` method is similar.
Again, we have to retrieve the payload from the HTTP request and use it for modifying the data in backend.
The difference is that case of update operation, the OData service is not expected to return any response payload. So we can skip the serialize-step and simply set the HTTP status code to **204 – no content**

```java
public void updateEntity(ODataRequest request, ODataResponse response, UriInfo uriInfo,
  ContentType requestFormat, ContentType responseFormat)
    throws ODataApplicationException, DeserializerException, SerializerException {

  // 1. Retrieve the entity set which belongs to the requested entity
  List<UriResource> resourcePaths = uriInfo.getUriResourceParts();
  // Note: only in our example we can assume that the first segment is the EntitySet
  UriResourceEntitySet uriResourceEntitySet = (UriResourceEntitySet) resourcePaths.get(0);
  EdmEntitySet edmEntitySet = uriResourceEntitySet.getEntitySet();
  EdmEntityType edmEntityType = edmEntitySet.getEntityType();

  // 2. update the data in backend
  // 2.1. retrieve the payload from the PUT request for the entity to be updated
  InputStream requestInputStream = request.getBody();
  ODataDeserializer deserializer = this.odata.createDeserializer(requestFormat);
  DeserializerResult result = deserializer.entity(requestInputStream, edmEntityType);
  Entity requestEntity = result.getEntity();
  // 2.2 do the modification in backend
  List<UriParameter> keyPredicates = uriResourceEntitySet.getKeyPredicates();
  // Note that this updateEntity()-method is invoked for both PUT or PATCH operations
  HttpMethod httpMethod = request.getMethod();
  storage.updateEntityData(edmEntitySet, keyPredicates, requestEntity, httpMethod);

  //3. configure the response object
  response.setStatusCode(HttpStatusCode.NO_CONTENT.getStatusCode());
}
```

In case of update, we have to consider the following:
The update of an entity can be realized in 2 ways: either a **PATCH** or a **PUT** request.
(See the online specification in section [11.4.3 Update an Entity](http://docs.oasis-open.org/odata/odata/v4.0/odata-v4.0-part1-protocol.html) for more details.
For both HTTP methods, our `updateEntity(...)` will be invoked.
But we have to treat the data-modification differently.
Therefore, we have to first retrieve the used HTTP method and in the backend-logic, we have to distinguish between **PATCH** and **PUT**.
The difference becomes relevant only in case if the user doesn’t send all the properties in the request body.

Example: if we modify the above example request body to look as follows:

```json
    {
      "Description":"updated description"
    }
```

Note that in this case, only one of three properties is sent in the request body.

  - If the HTTP method is **PATCH**:
    The value of the *Description* property is updated in the backend.
    The values of the other properties remain untouched.
  - If the HTTP method is **PUT**:
    The value of the *Description* property is updated in the backend.
    The value of the other properties is set to null (exception: key properties can never be null).


So let’s have a look at our sample implementation in the `Storage` class (see below for full sample code and also see the attached zip file containing the whole sample project)

```Java
private void updateProduct(EdmEntityType edmEntityType, List<UriParameter> keyParams, Entity entity, HttpMethod httpMethod)
                            throws ODataApplicationException{

  Entity productEntity = getProduct(edmEntityType, keyParams);
  if(productEntity == null){
    throw new ODataApplicationException("Entity not found",
                        HttpStatusCode.NOT_FOUND.getStatusCode(), Locale.ENGLISH);
  }

  // loop over all properties and replace the values with the values of the given payload
  // Note: ignoring ComplexType, as we don't have it in our odata model
  List<Property> existingProperties = productEntity.getProperties();
  for(Property existingProp : existingProperties){
    String propName = existingProp.getName();

    // ignore the key properties, they aren't updateable
    if(isKey(edmEntityType, propName)){
      continue;
    }

    Property updateProperty = entity.getProperty(propName);
    // the request payload might not consider ALL properties, so it can be null
    if(updateProperty == null){
      // if a property has NOT been added to the request payload
      // depending on the HttpMethod, our behavior is different
      if(httpMethod.equals(HttpMethod.PATCH)){
        // in case of PATCH, the existing property is not touched
        continue; // do nothing
      }else if(httpMethod.equals(HttpMethod.PUT)){
        // in case of PUT, the existing property is set to null
        existingProp.setValue(existingProp.getValueType(), null);
        continue;
      }
    }

    // change the value of the properties
    existingProp.setValue(existingProp.getValueType(), updateProperty.getValue());
  }
}
```

## 3.3. Implement the deleteEntity(...) method

In case of **DELETE** operation, the URL is the same like for the **GET** operation, but the request body is empty.

Example URL:

    http://localhost:8080/DemoService/DemoService.svc/Products(3)

The implementation is rather simple:

  - As usual, determine the entity set.
  - Delete the data in backend.
  - Configure the response object with the proper status code **204 – no content**.

```java
    public void deleteEntity(ODataRequest request, ODataResponse response, UriInfo uriInfo)
                            throws ODataApplicationException {

      // 1. Retrieve the entity set which belongs to the requested entity
      List<UriResource> resourcePaths = uriInfo.getUriResourceParts();
      // Note: only in our example we can assume that the first segment is the EntitySet
      UriResourceEntitySet uriResourceEntitySet = (UriResourceEntitySet) resourcePaths.get(0);
      EdmEntitySet edmEntitySet = uriResourceEntitySet.getEntitySet();

      // 2. delete the data in backend
      List<UriParameter> keyPredicates = uriResourceEntitySet.getKeyPredicates();
      storage.deleteEntityData(edmEntitySet, keyPredicates);

      //3. configure the response object
      response.setStatusCode(HttpStatusCode.NO_CONTENT.getStatusCode());
    }
```


# 4. Run the service

After building and deploying the project, we can invoke our OData service.

In order to test the write operations of our OData service, we need a tool that is able to execute the following required HTTP requests:

  - **POST**
  - **PUT**
  - **PATCH**
  - **DELETE**

This is usually done with any REST client tool that can be installed into the browser of your choice.

Some *REST* clients which are available as browser extension for:

  - Firefox: “RESTClient, a debugger for RESTful web services”
  - Chrome: “Advanced REST client”

The following sections provide examples for executing the requests:

### 4.1. Example for **CREATE**:

  - URL: <http://localhost:8080/DemoService/DemoService.svc/Products>
  - HTTP verb: **POST**
  - Header: `Content-Type: application/json; odata.metadata=minimal`
  - Request body:

```json
        {
          "ID":6,
          "Name":"Gamer Mouse",
          "Description":"optical mouse - gamer edition"
        }
```

**Note:** The value for the ID property is arbitrary, as it will be generated by our OData service implementation


### 4.2. Example for UPDATE (PUT):

  - URL: <http://localhost:8080/DemoService/DemoService.svc/Products(3)>
  - HTTP verb: **PUT**
  - Header: `Content-Type: application/json; odata.metadata=minimal`
  - Request body:

```json
        {
          "ID":3,
          "Name":"Ergo Screen updated Name",
          "Description":"updated description"
        }
```


### 4.3. Example for UPDATE (PATCH):

  - URL: <http://localhost:8080/DemoService/DemoService.svc/Products(3)>
  - HTTP verb: **PATCH**
  - Header: `Content-Type: application/json; odata.metadata=minimal`
  - Request body:

```json
        {
          "Description": "patched description"
        }
```

### 4.4. Example for DELETE:

  - URL: <http://localhost:8080/DemoService/DemoService.svc/Products(3)>
  - HTTP verb: **DELETE**
  - Header: Content-Type: application/json; odata.metadata=minimal
  - Request body:  `<empty>`

---

# 5. Summary

In this tutorial we have learned how to implement the creation, update and deletion of an entity.
It has been based on a simple OData model, focusing on simple sample code and sample data.

In the next tutorial (Part 4: Navigation) we will learn how to implement navigation, i.e. the linking of resources.

---

# 6. Links

### Tutorials
  * Tutorial OData V4 service part 1: [Read Entity Collection](/doc/odata4/tutorials/read/tutorial_read.html)
  * Tutorial OData V4 service part 2: [Read Entity, Read Property](/doc/odata4/tutorials/readep/tutorial_readep.html)
  * Tutorial OData V4 service part 3: Write (Create, Update, Delete Entity
  * Tutorial OData V4 service, part 4: [Navigation](/doc/odata4/tutorials/navigation/tutorial_navigation.html)
  * Tutorial OData V4 service, part 5.1: [System Query Options $top, $skip, $count (this page)](/doc/odata4/tutorials/sqo_tcs/tutorial_sqo_tcs.html)
  * Tutorial OData V4 service, part 5.2: [System Query Options $select, $expand](/doc/odata4/tutorials/sqo_es/tutorial_sqo_es.html)
  * Tutorial OData V4 service, part 5.3: [System Query Options $orderby](/doc/odata4/tutorials/sqo_o/tutorial_sqo_o.html)
  * Tutorial OData V4 service, part 5.4: [System Query Options $filter](/doc/odata4/tutorials/sqo_f/tutorial_sqo_f.html)
  * Tutorial ODATA V4 service, part 6: [Action and Function Imports](/doc/odata4/tutorials/action/tutorial_action.html)
  * Tutorial ODATA V4 service, part 7: [Media Entities](/doc/odata4/tutorials/media/tutorial_media.html)
  * Tutorial OData V4 service, part 8: [Batch Request support](/doc/odata4/tutorials/batch/tutorial_batch.html)
  * Tutorial OData V4 service, part 9: [Handling "Deep Insert" requests](/doc/odata4/tutorials/deep_insert/tutorial_deep_insert.html)
  
### Code and Repository
  * [Git Repository](https://gitbox.apache.org/repos/asf/olingo-odata4)
  * [Guide - To fetch the tutorial sources](/doc/odata4/tutorials/prerequisites/prerequisites.html)
  * [Demo Service source code as zip file (contains all tutorials)](http://www.apache.org/dyn/closer.lua/olingo/odata4/4.6.0/DemoService_Tutorial.zip)

### Further reading

  * [Official OData Homepage](http://odata.org/)
  * [OData documentation](http://www.odata.org/documentation/)
  * [Olingo Javadoc](/javadoc/odata4/index.html)
