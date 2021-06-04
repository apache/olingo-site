Title: How to build an OData Service with Olingo V4

# How to build an OData Service with Olingo V4

# Part 10: Bound Actions and Functions


**Table of Contents**

[TOC]

## Introduction

In the present tutorial, we’ll implement a bound action and function.

**Note:**
The final source code can be found in the project [git repository](https://gitbox.apache.org/repos/asf/olingo-odata4).
A detailed description how to checkout the tutorials can be found [here](/doc/odata4/tutorials/prerequisites/prerequisites.html).   
This tutorial can be found in subdirectory `/samples/tutorials/p9_action`

The [OData V4 specification](http://docs.oasis-open.org/odata/odata/v4.0/errata02/os/complete/part1-protocol/odata-v4.0-errata02-os-part1-protocol-complete.html#_Toc406398201) gives us a definition what *Functions*, *Actions* are:

> Operations allow the execution of custom logic on parts of a data
> model. Functions are operations that do not have side effects and may
> support further composition, for example, with additional filter
> operations, functions or an action. Actions are operations that allow
> side effects, such as data modification, and cannot be further
> composed in order to avoid non-deterministic behavior. Actions and
> functions are either bound to a type, enabling them to be called as
> members of an instance of that type, or unbound, in which case they
> are called as static operations. Action imports and function imports
> enable unbound actions and functions to be called from the service
> root.

In this short definition are several terms which are to be explained first. As you might expect operation is the superordinate of functions and actions. The result of operations can be:

 - An *entity* or a *collection of entities*
 - A *primitive property* or a *collection of primitive properties*
 - A *complex property* or a *collection of complex properties*

In addition an *Action* can return void that means there is no return value. A *Function* must return a value.

First an *Operation* can be bound or unbound.In this tutorial we will focus on bound operation. 

Bound actions and functions support overloading (multiple actions having the same name within the same namespace) by binding parameter type. The combination of action name and the binding parameter type MUST be unique within a namespace.

An action or a function element MAY specify a Boolean value for the IsBound attribute.
Actions/Functions whose IsBound attribute is false or not specified are considered unbound. Unbound actions/functions are invoked through an action import/function import.
Actions/Functions whose IsBound attribute is true are considered bound. Bound actions/functions are invoked by appending a segment containing the qualified action name to a segment of the appropriate binding parameter type within the resource path. Bound actions/functions MUST contain at least one edm:Parameter element, and the first parameter is the binding parameter. The binding parameter can be of any type, and it MAY be nullable.

Bound actions/functions that return an entity or a collection of entities MAY specify a value for the EntitySetPath attribute if determination of the entity set for the return type is contingent on the binding parameter. The value for the EntitySetPath attribute consists of a series of segments joined together with forward slashes. The first segment of the entity set path MUST be the name of the binding parameter. The remaining segments of the entity set path MUST represent navigation segments or type casts.

A navigation segment names the SimpleIdentifier of the navigation property to be traversed. A type cast segment names the QualifiedName of the entity type that should be returned from the type cast.

**Example**

For example there can be a bound action createOrders which is bound to the Customer entity having 2 parameters

Such an action can be expressed in the metadata document as follows

```xml
    <Action Name="CreateOrder" isBound=”true”>
     <Parameter Name="Customers" Type="SampleEntities.Customer" Nullable="false"/>
     <Parameter Name="quantity" Type="Edm.Int16" Nullable="false"/>
     <Parameter Name="discountCode" Type="Edm.String" Nullable="false"/>
     <ReturnType Type="Collection(SampleEntities.Orders)"/>
    </Action>
```

To call such a bound action the client issues a POST request to a URL identifying the action. In this simple case such a call could look like this:

      POST http://host/service/Customers('ALFKI')/SampleEntities.CreateOrder
      {
        "quantity": 2,
        "discountCode": "BLACKFRIDAY"
      }

Similarly there can be a bound function GetOrders which is bound to the Customer entity having 1 parameter

Such a function can be expressed in the metadata document as follows

```xml
    <Function Name="GetOrders" isBound=”true”>
     <Parameter Name="Customers" Type="SampleEntities.Customer" Nullable="false"/>
     <Parameter Name="discountCode" Type="Edm.String" Nullable="false"/>
     <ReturnType Type="Collection(SampleEntities.Orders)"/>
    </Function>
```

To call such a bound function the client issues a GET request to a URL identifying the function. In this simple case such a call could look like this:

      GET http://host/service/Customers('ALFKI')/SampleEntities.GetOrders(discountCode='BLACKFRIDAY')
     

## Preparation

You should read the previous tutorials first to have an idea how to read entities and entity collections. 

As a shortcut you should checkout the prepared tutorial project in the [git repository](https://gitbox.apache.org/repos/asf/olingo-odata4) in folder /samples/tutorials/p9_action_preparation.

Afterwards do a Deploy and run: it should be working. At this state you can perform CRUD operations and do navigations between products and categories.

## Implementation

We use the given data model you are familiar with. To keep things simple we implement one bound action and one bound function.

**Bound Action that returns a collection of entities: DiscountProducts**    
This action takes bound parameter “*ParamCategory*” and an additional parameter “*Amount*”. The action updates the price of all products related to categories by applying the discount amount.

After finishing the implementation the definition of the action should be like this:

```xml
    <Action Name="DiscountProducts" IsBound="true">
      <Parameter Name="ParamCategory" Type="Collection(OData.Demo.Category)"/>
      <Parameter Name="Amount" Type="Edm.Int32"/>
      <ReturnType Type="Collection(OData.Demo.Product)"/>
    </Action>
```

**Bound Action that returns an entity: DiscountProduct**

This action takes bound parameter “*ParamCategory*” and an additional parameter “*Amount*”. The actions updates the price of a specific products related to a category by applying the discount amount.
After finishing the implementation the definition of the action should be like this:

```xml
    <Action Name="DiscountProduct" IsBound="true">
       <Parameter Name="ParamCategory" Type=”OData.Demo.Category"/>
       <Parameter Name="Amount" Type="Edm.Int32"/>
       <ReturnType Type=" OData.Demo.Product"/>
    </Action>
```

While actions are called by using HTTP Method POST is nessesary to introduce new processor interfaces for actions. So there exists a bunch of interfaces, for each return type strictly one.


**Bound Function that returns a collection of entities: GetDiscountedProducts**    
This function takes bound parameter “*ParamCategory*” and an additional parameter “*Amount*”. The function lists all the products related to categories which are eligible for the discount amount.

After finishing the implementation the definition of the action should be like this:

```xml
    <Function Name="GetDiscountedProducts" IsBound="true">
      <Parameter Name="ParamCategory" Type="Collection(OData.Demo.Category)"/>
      <Parameter Name="Amount" Type="Edm.Int32"/>
      <ReturnType Type="Collection(OData.Demo.Product)"/>
    </Function>
```

**Bound Function that returns an entity: GetDiscountedProduct**

This function takes bound parameter “*ParamCategory*” and an additional parameter “*Amount*”. The function lists one specific product related to a category which is eligible for the discount amount.
After finishing the implementation the definition of the action should be like this:

```xml
    <Function Name="GetDiscountedProduct" IsBound="true">
       <Parameter Name="ParamCategory" Type=”OData.Demo.Category"/>
       <Parameter Name="Amount" Type="Edm.Int32"/>
       <ReturnType Type=" OData.Demo.Product"/>
    </Function>
```

**Steps**    

  * Extend the Metadata model
  * Extend the data store
  * Implement an action processor

### Extend the Metadata model

Create the following constants in the DemoEdmProvider. These constants are used to address the actions.

```java
    //Bound Action
    public static final String ACTION_PROVIDE_DISCOUNT = "DiscountProducts";
    public static final FullQualifiedName ACTION_PROVIDE_DISCOUNT_FQN = new FullQualifiedName(NAMESPACE, ACTION_PROVIDE_DISCOUNT);

    public static final String ACTION_PROVIDE_DISCOUNT_FOR_PRODUCT = "DiscountProduct";
    public static final FullQualifiedName ACTION_PROVIDE_DISCOUNT_FOR_PRODUCT_FQN = new FullQualifiedName(NAMESPACE, ACTION_PROVIDE_DISCOUNT_FOR_PRODUCT);
    
    //Bound Function
    public static final String FUNCTION_PROVIDE_DISCOUNT = "GetDiscountedProducts";
    public static final FullQualifiedName FUNCTION_PROVIDE_DISCOUNT_FQN = new FullQualifiedName(NAMESPACE, FUNCTION_PROVIDE_DISCOUNT);
      
    public static final String FUNCTION_PROVIDE_DISCOUNT_FOR_PRODUCT = "GetDiscountedProduct";
    public static final FullQualifiedName FUNCTION_PROVIDE_DISCOUNT_FOR_PRODUCT_FQN = new FullQualifiedName(NAMESPACE, FUNCTION_PROVIDE_DISCOUNT_FOR_PRODUCT);
    
    //Parameters
    public static final String PARAMETER_AMOUNT = "Amount";
    
    //Binding Parameter
    public static final String PARAMETER_CATEGORY = "ParamCategory";
```

The way to announce the operations is very similar to announcing EntityTypes. We have to override some methods. Those methods provide the definition of the Edm elements. We need methods for:

  - Actions
  - Functions

The code is simple and straight forward. We need to create a list of parameters of which the first parameter should be the binding parameter, then create a return type. At the end all parts are fit together and get returned as new CsdlAction Object.

```java
        @Override
        public List<CsdlAction> getActions(final FullQualifiedName actionName) {
       // It is allowed to overload actions, so we have to provide a list of Actions for each action name
        final List<CsdlAction> actions = new ArrayList<CsdlAction>();
        
        if (actionName.equals(ACTION_PROVIDE_DISCOUNT_FQN)) {
          // Create parameters
          final List<CsdlParameter> parameters = new ArrayList<CsdlParameter>();
          CsdlParameter parameter = new CsdlParameter();
          parameter.setName(PARAMETER_CATEGORY);
          parameter.setType(ET_CATEGORY_FQN);
          parameter.setCollection(true);
          parameters.add(parameter);
          parameter = new CsdlParameter();
          parameter.setName(PARAMETER_AMOUNT);
          parameter.setType(EdmPrimitiveTypeKind.Int32.getFullQualifiedName());
          parameters.add(parameter);
          
          // Create the Csdl Action
          final CsdlAction action = new CsdlAction();
          action.setName(ACTION_PROVIDE_DISCOUNT_FQN.getName());
          action.setBound(true);
          action.setParameters(parameters);
          action.setReturnType(new CsdlReturnType().setType(ET_PRODUCT_FQN).setCollection(true));
          actions.add(action);
          
          return actions;
        } else if (actionName.equals(ACTION_PROVIDE_DISCOUNT_FOR_PRODUCT_FQN)) {
          // Create parameters
          final List<CsdlParameter> parameters = new ArrayList<CsdlParameter>();
          CsdlParameter parameter = new CsdlParameter();
          parameter.setName(PARAMETER_CATEGORY);
          parameter.setType(ET_CATEGORY_FQN);
          parameter.setCollection(false);
          parameters.add(parameter);
          parameter = new CsdlParameter();
          parameter.setName(PARAMETER_AMOUNT);
          parameter.setType(EdmPrimitiveTypeKind.Int32.getFullQualifiedName());
          parameters.add(parameter);
          
          // Create the Csdl Action
          final CsdlAction action = new CsdlAction();
          action.setName(ACTION_PROVIDE_DISCOUNT_FOR_PRODUCT_FQN.getName());
          action.setBound(true);
          action.setParameters(parameters);
          action.setReturnType(new CsdlReturnType().setType(ET_PRODUCT_FQN).setCollection(false));
          actions.add(action);
          
          return actions;
        }
        
        return null;
      }
```

Similarly, for functions we need to create a list of parameters of which the first parameter should be the binding parameter, then create a return type. At the end all parts are fit together and get returned as new CsdlFunction Object.

```java
        @Override
        public List<CsdlFunction> getFunctions(final FullQualifiedName functionName) {
       // It is allowed to overload functions, so we have to provide a list of Functions for each function name
        final List<CsdlFunction> functions = new ArrayList<CsdlFunction>();
        
        if (functionName.equals(FUNCTION_PROVIDE_DISCOUNT_FQN)) {
          // Create parameters
          final List<CsdlParameter> parameters = new ArrayList<CsdlParameter>();
          CsdlParameter parameter = new CsdlParameter();
          parameter.setName(PARAMETER_CATEGORY);
          parameter.setType(ET_CATEGORY_FQN);
          parameter.setCollection(true);
          parameters.add(parameter);
          parameter = new CsdlParameter();
          parameter.setName(PARAMETER_AMOUNT);
          parameter.setType(EdmPrimitiveTypeKind.Int32.getFullQualifiedName());
          parameters.add(parameter);
          
          // Create the Csdl Function
          final CsdlFunction function = new CsdlFunction();
          function.setName(FUNCTION_PROVIDE_DISCOUNT_FQN.getName());
          function.setBound(true);
          function.setParameters(parameters);
          function.setReturnType(new CsdlReturnType().setType(ET_PRODUCT_FQN).setCollection(true));
          functions.add(function);
          
          return functions;
        } else if (functionName.equals(FUNCTION_PROVIDE_DISCOUNT_FOR_PRODUCT_FQN)) {
          // Create parameters
          final List<CsdlParameter> parameters = new ArrayList<CsdlParameter>();
          CsdlParameter parameter = new CsdlParameter();
          parameter.setName(PARAMETER_CATEGORY);
          parameter.setType(ET_CATEGORY_FQN);
          parameter.setCollection(false);
          parameters.add(parameter);
          parameter = new CsdlParameter();
          parameter.setName(PARAMETER_AMOUNT);
          parameter.setType(EdmPrimitiveTypeKind.Int32.getFullQualifiedName());
          parameters.add(parameter);
          
          // Create the Csdl Function
          final CsdlFunction function= new CsdlFunction();
          function.setName(ACTION_PROVIDE_DISCOUNT_FOR_PRODUCT_FQN.getName());
          function.setBound(true);
          function.setParameters(parameters);
          function.setReturnType(new CsdlReturnType().setType(ET_PRODUCT_FQN).setCollection(false));
          functions.add(function);
          
          return functions;
        }
        
        return null;
      }
```

Finally we have to announce these operations to the schema. Add the following lines to the method getSchemas():

```java
    // add actions
    List<CsdlAction> actions = new ArrayList<CsdlAction>();
    actions.addAll(getActions(ACTION_PROVIDE_DISCOUNT_FQN));
    actions.addAll(getActions(ACTION_PROVIDE_DISCOUNT_FOR_PRODUCT_FQN));
    schema.setActions(actions);

    // add functions
    List<CsdlFunction> functions = new ArrayList<CsdlFunction>();
    functions.addAll(getFunctions(FUNCTION_PROVIDE_DISCOUNT_FQN));
    functions.addAll(getFunctions(FUNCTION_PROVIDE_DISCOUNT_FOR_PRODUCT_FQN));
    schema.setFunctions(functions);
```

### Extend the data store

We need two methods in the data store to read the action DiscountProducts and DiscountProduct. The first method returns a collection of entites and second method returns a single entity.


```java
        public EntityCollection processBoundActionEntityCollection(EdmAction action, Map<String, Parameter> parameters) {
        EntityCollection collection = new EntityCollection();
        if ("DiscountProducts".equals(action.getName())) {
          for (Entity entity : categoryList) {
            Entity en = getRelatedEntity(entity, (EdmEntityType) action.getReturnType().getType());
            Integer currentValue = (Integer)en.getProperty("Price").asPrimitive();
            Integer newValue = currentValue - (Integer)parameters.get("Amount").asPrimitive();
            en.getProperty("Price").setValue(ValueType.PRIMITIVE, newValue);
            collection.getEntities().add(en);
          }
        }
        return collection;
      }
    
      public DemoEntityActionResult processBoundActionEntity(EdmAction action, Map<String, Parameter> parameters,
          List<UriParameter> keyParams) throws ODataApplicationException {
        DemoEntityActionResult result = new DemoEntityActionResult();
        if ("DiscountProduct".equals(action.getName())) {
          for (Entity entity : categoryList) {
            Entity en = getRelatedEntity(entity, (EdmEntityType) action.getReturnType().getType(), keyParams);
            Integer currentValue = (Integer)en.getProperty("Price").asPrimitive();
            Integer newValue = currentValue - (Integer)parameters.get("Amount").asPrimitive();
            en.getProperty("Price").setValue(ValueType.PRIMITIVE, newValue);
            result.setEntity(en);
            result.setCreated(true);
            return result;
          }
        }
        return null;
      }
```

In the second method, we are returning a custom object DemoEntityActionResult. This holds the entity and the status as to whether the entity is created or just returned. This information is used to set the response status.

```java
      public class DemoEntityActionResult {
      private Entity entity;
      private boolean created = false;
    
      public Entity getEntity() {
        return entity;
      }
    
      public DemoEntityActionResult setEntity(final Entity entity) {
        this.entity = entity;
        return this;
      }
    
      public boolean isCreated() {
        return created;
      }
    
      public DemoEntityActionResult setCreated(final boolean created) {
        this.created = created;
        return this;
      }
    }
```    

We also create methods for GetDiscountedProducts and GetDiscountedProduct functions. 

```java
        public EntityCollection getBoundFunctionEntityCollection(EdmFunction function, Integer amount) {
        EntityCollection collection = new EntityCollection();
        if ("GetDiscountedProducts".equals(function.getName())) {
          for (Entity entity : categoryList) {
            if(amount >= entity.getProperty("amount")){
              Entity en = getRelatedEntity(entity, (EdmEntityType) function.getReturnType().getType());
              collection.getEntities().add(en);
            }
          }
        }
        return collection;
      }
    
      public Entity getBoundFunctionEntity(EdmAction function, Integer amount) throws ODataApplicationException {
        if ("GetDiscountedProduct".equals(function.getName())) {
          for (Entity entity : categoryList) {
            if(amount== entity.getProperty("amount")){
              return getRelatedEntity(entity, (EdmEntityType) function.getReturnType().getType(), keyParams);
            }
          }
        }
        return null;
      }
```

### Extend the entity collection and the entity processor to handle functions

We start with the entity collection processor DemoEntityCollectionProcessor. To keep things simple, the first steps is to distinguish between entity collections and function imports. A cleverer implementation can handle both cases in one method to avoid duplicated code.

The recent implementation of the readEntityCollection() has been moved to readEntityCollectionInternal()

```java
public void readEntityCollection(ODataRequest request, ODataResponse response, UriInfo uriInfo, ContentType responseFormat) throws ODataApplicationException, SerializerException {

  final UriResource firstResourceSegment = uriInfo.getUriResourceParts().get(0);

  if(firstResourceSegment instanceof UriResourceEntitySet) {
    readEntityCollectionInternal(request, response, uriInfo, responseFormat);
  } else if(firstResourceSegment instanceof UriResourceFunction) {
    readFunctionImportCollection(request, response, uriInfo, responseFormat);
  } else {
    throw new ODataApplicationException("Not implemented",
      HttpStatusCode.NOT&#95;IMPLEMENTED.getStatusCode(),
    Locale.ENGLISH);
  }
}
```

Like by reading entity collections, the first step is to analyze the URI and then fetch the data (of the function).

```java
     private void readEntityCollectionInternal(final ODataRequest request, final ODataResponse response,
     final UriInfo uriInfo, final ContentType responseFormat) throws ODataApplicationException, SerializerException {

     EdmEntitySet responseEdmEntitySet = null; // we'll need this to build the ContextURL
     EntityCollection responseEntityCollection = null; // we'll need this to set the response body

     // 1st retrieve the requested EntitySet from the uriInfo (representation of the parsed URI)
     List<UriResource> resourceParts = uriInfo.getUriResourceParts();
     int segmentCount = resourceParts.size();

     UriResource uriResource = resourceParts.get(0); // in our example, the first segment is the EntitySet
     if (!(uriResource instanceof UriResourceEntitySet)) {
       throw new ODataApplicationException("Only EntitySet is supported",
           HttpStatusCode.NOT_IMPLEMENTED.getStatusCode(), Locale.ROOT);
     }

     UriResourceEntitySet uriResourceEntitySet = (UriResourceEntitySet) uriResource;
     EdmEntitySet startEdmEntitySet = uriResourceEntitySet.getEntitySet();

      if (segmentCount == 1) { 
      // This is a normal query fetch the entity from backend and return entityset
      }

      else if (segmentCount == 2) { // in case of function or navigation

        UriResource lastSegment = resourceParts.get(1); // in our example we don't support more complex URIs
        if (lastSegment instanceof UriResourceFunction) {// For bound function
        UriResourceFunction uriResourceFunction = (UriResourceFunction) lastSegment;
        // 2nd: fetch the data from backend
        // first fetch the target entity type 
        String targetEntityType = uriResourceFunction.getFunction().getReturnType().getType().getName();
        // contextURL displays the last segment
        for(EdmEntitySet entitySet : serviceMetadata.getEdm().getEntityContainer().getEntitySets()){
          if(targetEntityType.equals(entitySet.getEntityType().getName())){
            responseEdmEntitySet = entitySet;
            break;
          }
        }
        
        // error handling for null entities
        if (targetEntityType == null || responseEdmEntitySet == null) {
          throw new ODataApplicationException("Entity not found.",
              HttpStatusCode.NOT_FOUND.getStatusCode(), Locale.ROOT);
        }
        Integer amount = Integer.parseInt(uriResourceFunction.getParameters().get(0).getText())
        // then fetch the entity collection for the target type
        responseEntityCollection = storage.getBoundFunctionEntityCollection(function, amount);
      }
    }
      // 3rd: create and configure a serializer
     ContextURL contextUrl = ContextURL.with().entitySet(responseEdmEntitySet).build();
     final String id = request.getRawBaseUri() + "/" + responseEdmEntitySet.getName();
     EntityCollectionSerializerOptions opts = EntityCollectionSerializerOptions.with()
        .contextURL(contextUrl).id(id).build();
     EdmEntityType edmEntityType = responseEdmEntitySet.getEntityType();

     ODataSerializer serializer = odata.createSerializer(responseFormat);
     SerializerResult serializerResult = serializer.entityCollection(serviceMetadata, edmEntityType,
        responseEntityCollection, opts);

     // 4th: configure the response object: set the body, headers and status code
     response.setContent(serializerResult.getContent());
     response.setStatusCode(HttpStatusCode.OK.getStatusCode());
     response.setHeader(HttpHeader.CONTENT_TYPE, responseFormat.toContentTypeString());
    }
```


Next we will implement the processor to read a single entity. The implementation is quite similar to the implementation of the collection processor.

```java
    public void readEntity(ODataRequest request, ODataResponse response, UriInfo uriInfo, ContentType responseFormat)
      throws ODataApplicationException, SerializerException {

      UriResource uriResource = uriInfo.getUriResourceParts().get(0);

      if(uriResource instanceof UriResourceEntitySet) {
        readEntityInternal(request, response, uriInfo, responseFormat);
      } else if(uriResource instanceof UriResourceFunction) {
        readFunctionImportInternal(request, response, uriInfo, responseFormat);
      } else {
        throw new ODataApplicationException("Only EntitySet is supported",
          HttpStatusCode.NOT&#95;IMPLEMENTED.getStatusCode(), Locale.ENGLISH);
      }
    }

     private void readEntityInternal(final ODataRequest request, final ODataResponse response,
      final UriInfo uriInfo, final ContentType responseFormat) throws ODataApplicationException, SerializerException {

       EdmEntityType responseEdmEntityType = null; // we'll need this to build the ContextURL
       Entity responseEntity = null; // required for serialization of the response body
       EdmEntitySet responseEdmEntitySet = null; // we need this for building the contextUrl

       // 1st step: retrieve the requested Entity: can be "normal" read operation, or navigation (to-one)
       List<UriResource> resourceParts = uriInfo.getUriResourceParts();
       int segmentCount = resourceParts.size();
    
       UriResource uriResource = resourceParts.get(0); // in our example, the first segment is the EntitySet
       UriResourceEntitySet uriResourceEntitySet = (UriResourceEntitySet) uriResource;
       EdmEntitySet startEdmEntitySet = uriResourceEntitySet.getEntitySet();

      if (segmentCount == 1) { 
      // This is a normal read call fetch the entity from backend and return entity
      } else if (segmentCount == 2) { // Bound Function or navigation
        UriResource segment = resourceParts.get(1);
        if (segment instanceof UriResourceFunction) {
          UriResourceFunction uriResourceFunction = (UriResourceFunction) segment;

          // 2nd: fetch the data from backend.
          // first fetch the target entity type 
          String targetEntityType = uriResourceFunction.getFunction().getReturnType().getType().getName();
       
          // contextURL displays the last segment
          for(EdmEntitySet entitySet : serviceMetadata.getEdm().getEntityContainer().getEntitySets()){
            if(targetEntityType.equals(entitySet.getEntityType().getName())){
              responseEdmEntityType = entitySet.getEntityType();
              responseEdmEntitySet = entitySet;
              break;
            }
          }
        
          // error handling for null entities
          if (targetEntityType == null || responseEdmEntitySet == null) {
            throw new ODataApplicationException("Entity not found.",
                HttpStatusCode.NOT_FOUND.getStatusCode(), Locale.ROOT);
          }

        Integer amount = Integer.parseInt(uriResourceFunction.getParameters().get(0).getText())
        // then fetch the entity collection for the target type
        responseEntity = storage.getBoundFunctionEntity(function, amount);
      }
    }

    if (responseEntity == null) {
      // this is the case for e.g. DemoService.svc/Categories(4) or DemoService.svc/Categories(3)/Products(999)
      throw new ODataApplicationException("Nothing found.", HttpStatusCode.NOT_FOUND.getStatusCode(), Locale.ROOT);
    }

    // 3. serialize
    ContextURL contextUrl = ContextURL.with().entitySet(responseEdmEntitySet).suffix(Suffix.ENTITY).build();
    EntitySerializerOptions opts = EntitySerializerOptions.with().contextURL(contextUrl).build();

    ODataSerializer serializer = odata.createSerializer(responseFormat);
    SerializerResult serializerResult = serializer.entity(serviceMetadata,
        responseEdmEntityType, responseEntity, opts);

    // 4. configure the response object
    response.setContent(serializerResult.getContent());
    response.setStatusCode(HttpStatusCode.OK.getStatusCode());
    response.setHeader(HttpHeader.CONTENT_TYPE, responseFormat.toContentTypeString());
    }
```

### Implement an action processor

Create a new class `DemoActionProcessor` make them implement the interface interface 'ActionEntityCollectionProcessor' and 'ActionEntityProcessor'.

```java
      public class DemoActionProcessor implements ActionEntityCollectionProcessor, ActionEntityProcessor {
    
      private OData odata;
      private Storage storage;
      private ServiceMetadata serviceMetadata;
    
      public DemoActionProcessor(final Storage storage) {
        this.storage = storage;
      }
    
      @Override
      public void init(final OData odata, final ServiceMetadata serviceMetadata)   {
        this.odata = odata;
        this.serviceMetadata = serviceMetadata;
      }
```

The first overriden method returns a collection of entities.

First analyze the uri. Bound Actions will have the first segment in the resource path to be an entity set. It can then be followed by a navigation segment or a type cast. The last segment will be the fully qualified action name. 

Then deserialize the action parameters.

Execute the action and set the response code.

```java

        @Override
      public void processActionEntityCollection(ODataRequest request, ODataResponse response, UriInfo uriInfo,
          ContentType requestFormat, ContentType responseFormat) throws ODataApplicationException, ODataLibraryException {
    
        Map<String, Parameter> parameters = new HashMap<String, Parameter>();
        EdmAction action = null;
        EntityCollection collection = null;
        
        if (requestFormat == null) {
          throw new ODataApplicationException("The content type has not been set in the request.",
              HttpStatusCode.BAD_REQUEST.getStatusCode(), Locale.ROOT);
        }
        
        List<UriResource> resourcePaths = uriInfo.asUriInfoResource().getUriResourceParts();
        final ODataDeserializer deserializer = odata.createDeserializer(requestFormat);
        UriResourceEntitySet boundEntitySet = (UriResourceEntitySet) resourcePaths.get(0);
        if (resourcePaths.size() > 1) {
    	// Check if there is a navigation segment added after the bound parameter
          if (resourcePaths.get(1) instanceof UriResourceAction) {
           action = ((UriResourceAction) resourcePaths.get(2))
                .getAction();
            throw new ODataApplicationException("Action " + action.getName() + " is not yet implemented.",
            HttpStatusCode.NOT_IMPLEMENTED.getStatusCode(), Locale.ENGLISH);      } else {
            action = ((UriResourceAction) resourcePaths.get(1))
                .getAction();
            parameters = deserializer.actionParameters(request.getBody(), action)
                .getActionParameters();
            collection =
                storage.processBoundActionEntityCollection(action, parameters);
          }
        }
        // Collections must never be null.
        // Not nullable return types must not contain a null value.
        if (collection == null
            || collection.getEntities().contains(null) && !action.getReturnType().isNullable()) {
          throw new ODataApplicationException("The action could not be executed.",
              HttpStatusCode.INTERNAL_SERVER_ERROR.getStatusCode(), Locale.ROOT);
        }
    
        final Return returnPreference = odata.createPreferences(request.getHeaders(HttpHeader.PREFER)).getReturn();
        if (returnPreference == null || returnPreference == Return.REPRESENTATION) {
          final EdmEntitySet edmEntitySet = boundEntitySet.getEntitySet();
          final EdmEntityType type = (EdmEntityType) action.getReturnType().getType();
          final EntityCollectionSerializerOptions options = EntityCollectionSerializerOptions.with()
              .contextURL(isODataMetadataNone(responseFormat) ? null : getContextUrl(action.getReturnedEntitySet(edmEntitySet), type, false))
              .build();
          response.setContent(odata.createSerializer(responseFormat)
              .entityCollection(serviceMetadata, type, collection, options).getContent());
          response.setHeader(HttpHeader.CONTENT_TYPE, responseFormat.toContentTypeString());
          response.setStatusCode(HttpStatusCode.OK.getStatusCode());
        } else {
          response.setStatusCode(HttpStatusCode.NO_CONTENT.getStatusCode());
        }
        if (returnPreference != null) {
          response.setHeader(HttpHeader.PREFERENCE_APPLIED,
              PreferencesApplied.with().returnRepresentation(returnPreference).build().toValueString());
        }
      }
      
    //This method fetches the context URL
      private ContextURL getContextUrl(final EdmEntitySet entitySet, final EdmEntityType entityType,
          final boolean isSingleEntity) throws ODataLibraryException {
        Builder builder = ContextURL.with();
        builder = entitySet == null ?
            isSingleEntity ? builder.type(entityType) : builder.asCollection().type(entityType) :
            builder.entitySet(entitySet);
        builder = builder.suffix(isSingleEntity && entitySet != null ? Suffix.ENTITY : null);
        return builder.build();
      }
      
      protected boolean isODataMetadataNone(final ContentType contentType) {
        return contentType.isCompatible(ContentType.APPLICATION_JSON)
            && ContentType.VALUE_ODATA_METADATA_NONE.equalsIgnoreCase(
                contentType.getParameter(ContentType.PARAMETER_ODATA_METADATA));
      }
```

The second method to be overriden returns a single entity.

Again first analyze the uri. Bound Actions will have the first segment in the resource path to be an entity set with a key predicate. It can then be followed by a navigation segment or a type cast. The last segment will be the fully qualified action name. 

Then deserialize the action parameters.

Execute the action and set the response code.

```java

        @Override
      public void processActionEntity(ODataRequest request, ODataResponse response, UriInfo uriInfo,
          ContentType requestFormat, ContentType responseFormat) throws ODataApplicationException, ODataLibraryException {
    
        EdmAction action = null;
        Map<String, Parameter> parameters = new HashMap<String, Parameter>(); 
       // DemoEntityActionResult is a custom object that holds the entity and the status as to whether the entity is created or just returned. This information is used to set the response status
        DemoEntityActionResult entityResult = null;
        if (requestFormat == null) {
          throw new ODataApplicationException("The content type has not been set in the request.",
              HttpStatusCode.BAD_REQUEST.getStatusCode(), Locale.ROOT);
        }
        
        final ODataDeserializer deserializer = odata.createDeserializer(requestFormat);
        final List<UriResource> resourcePaths = uriInfo.asUriInfoResource().getUriResourceParts();
        UriResourceEntitySet boundEntity = (UriResourceEntitySet) resourcePaths.get(0);
        if (resourcePaths.size() > 1) {
    	// Checks if there is a navigation segment added after the binding parameter
          if (resourcePaths.get(1) instanceof UriResourceAction) {
            action = ((UriResourceAction) resourcePaths.get(1))
                .getAction();
            throw new ODataApplicationException("Action " + action.getName() + " is not yet implemented.",
            HttpStatusCode.NOT_IMPLEMENTED.getStatusCode(), Locale.ENGLISH);
          } else if (resourcePaths.get(0) instanceof UriResourceEntitySet) {
            action = ((UriResourceAction) resourcePaths.get(1))
                .getAction();
            parameters = deserializer.actionParameters(request.getBody(), action)
                .getActionParameters();
            entityResult =
                storage.processBoundActionEntity(action, parameters, boundEntity.getKeyPredicates());
          }
        }
        final EdmEntitySet edmEntitySet = boundEntity.getEntitySet();
        final EdmEntityType type = (EdmEntityType) action.getReturnType().getType();
    
        if (entityResult == null || entityResult.getEntity() == null) {
          if (action.getReturnType().isNullable()) {
            response.setStatusCode(HttpStatusCode.NO_CONTENT.getStatusCode());
          } else {
            // Not nullable return type so we have to give back a 500
            throw new ODataApplicationException("The action could not be executed.",
                HttpStatusCode.INTERNAL_SERVER_ERROR.getStatusCode(), Locale.ROOT);
          }
        } else {
          final Return returnPreference = odata.createPreferences(request.getHeaders(HttpHeader.PREFER)).getReturn();
          if (returnPreference == null || returnPreference == Return.REPRESENTATION) {
            response.setContent(odata.createSerializer(responseFormat).entity(
                serviceMetadata,
                type,
                entityResult.getEntity(),
                EntitySerializerOptions.with()
                    .contextURL(isODataMetadataNone(responseFormat) ? null : getContextUrl(action.getReturnedEntitySet(edmEntitySet), type, true))
                    .build())
                .getContent());
            response.setHeader(HttpHeader.CONTENT_TYPE, responseFormat.toContentTypeString());
            response.setStatusCode((entityResult.isCreated() ? HttpStatusCode.CREATED : HttpStatusCode.OK)
                .getStatusCode());
          } else {
            response.setStatusCode(HttpStatusCode.NO_CONTENT.getStatusCode());
          }
          if (returnPreference != null) {
            response.setHeader(HttpHeader.PREFERENCE_APPLIED,
                PreferencesApplied.with().returnRepresentation(returnPreference).build().toValueString());
          }
          if (entityResult.isCreated()) {
            final String location = request.getRawBaseUri() + '/'
                + odata.createUriHelper().buildCanonicalURL(edmEntitySet, entityResult.getEntity());
            response.setHeader(HttpHeader.LOCATION, location);
            if (returnPreference == Return.MINIMAL) {
              response.setHeader(HttpHeader.ODATA_ENTITY_ID, location);
            }
          }
          if (entityResult.getEntity().getETag() != null) {
            response.setHeader(HttpHeader.ETAG, entityResult.getEntity().getETag());
          }
        }    
      }
```

## Run the implemented service

After building and deploying your service to your server, you can try the following requests:

**Functions (Called via GET)**     

  * [http://localhost:8080/DemoService-Action/DemoService.svc/Categories/OData.Demo.GetDiscountedProducts(Amount=50)](http://localhost:8080/DemoService-Action/DemoService.svc/Categories/OData.Demo.GetDiscountedProducts(Amount=50))


  * [http://localhost:8080/DemoService-Action/DemoService.svc/Categories(0)/OData.Demo.GetDiscountedProduct(Amount=50)](http://localhost:8080/DemoService-Action/DemoService.svc/Categories(0)/OData.Demo.GetDiscountedProduct(Amount=50))

**Actions (Called via POST)**     
*Note:* Set the Content-Type header to: `Content-Type: application/json`

  * [http://localhost:8080/DemoService-Action/DemoService.svc/Categories/OData.Demo.DiscountProducts](http://localhost:8080/DemoService-Action/DemoService.svc/Categories/OData.Demo.DiscountProducts)

    Content:

    {"Amount":50 }


  * [http://localhost:8080/DemoService-Action/DemoService.svc/Categories(0)/OData.Demo.DiscountProduct](http://localhost:8080/DemoService-Action/DemoService.svc/Categories(0)/OData.Demo.DiscountProduct)

    Content:

    { "Amount": 50 }


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
  * Tutorial ODATA V4 service, part 6: Action and Function Imports
  * Tutorial ODATA V4 service, part 7: [Media Entities](/doc/odata4/tutorials/media/tutorial_media.html)
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
