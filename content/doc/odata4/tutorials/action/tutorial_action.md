Title: How to build an OData Service with Olingo V4

# How to build an OData Service with Olingo V4

# Part 6: Action Imports and Function Imports


**Table of Contents**

[TOC]

## Introduction

In the present tutorial, we’ll implement a function import and an action import as well.

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

The definition gives us some parts where function and actions can be used. First an *Operation* can be bound or unbound. In this tutorial we will focus on unbound operations. Unbound operations are something like static methods in Java, so if one of these operations have parameters we have to pass all of them explicit to the operation.


**Example**

For example there could be a function that calculates the VAT. The result depends on the one hand from the net price and on the other hand from the country in which the customer lives.


Such a function can be expressed in the metadata document as follows

~~~xml
    <Function Name="CalculateVAT">
         <Parameter Name="NetPrice" Type="Edm.Decimal" Nullable="false"/>
         <Parameter Name="Country" Type="Edm.String" Nullable="false"/>

        <ReturnType Type="Edm.Decimal"/>
    </Function>
~~~

To make this function statically callable we have to define a Function Import.

~~~xml
    <EntityContainer Name="Container">
         <FunctionImport Name="StaticCalculateVAT"
                         Function="CalculateVAT"
                         IncludeInServiceDocument="true"/>
    </EntityContainer>
~~~

To call such a Function Import the client issues a GET requests to a URL identifying the function import. The parameters are passed by using the so called inline parameter syntax. In this simple case such a call could look like this:

    http://host/myService/StaticCalculateVAT(NetPrice=123.00,Country=’US’)

The definition talks about composing of functions. By default every function is Composable=false, that means there must be no further resource parts after the function call and there must also be no system query options. If a function is composable you are be able to use system query options. Which options are allowed in particular is based on the return type of the function. Further you can navigate to properties and use Navigation Properties to navigate to related entities as well.

The definition of Actions / Action Imports in metadata document is similar to functions.

~~~xml
    <Action Name="Reset">
        <Parameter Name="Amount" Type="Edm.Int32"/>
    </Action>

    <EntityContainer Name="Container">
        <ActionImport Name="StaticReset" Action="Reset"/>
    </EntityContainer>
~~~

As you can see, this function does not return any value and takes one parameter “*Amount*” To call this action import, you have to issue an POST request to

    http://host/myService/StaticReset

The parameters are passed within the body of the request. In this case such a body could look like the following (JSON - Syntax):

~~~json
    {
      "Amount": 2
    }
~~~

As you read in the definition, actions can have side effects (modifying the data) but cannot be composed like functions.

## Preparation

You should read the previous tutorials first to have an idea how to read entities and entity collections. In addition the following code is based on the write tutorial merged with the navigation tutorial.

As a shortcut you should checkout the prepared tutorial project in the [git repository](https://gitbox.apache.org/repos/asf/olingo-odata4) in folder /samples/tutorials/p9_action_preparation.

Afterwards do a Deploy and run: it should be working. At this state you can perform CRUD operations and do navigations between products and categories.

## Implementation

We use the given data model you are familiar with. To keep things simple we implement one function import and one action import.

**Function Import: CountCategories**    
This function takes a mandatory parameter “*Amount*”. The function returns a collection of categories with the very same number of related products.

After finishing the implementation the definition of the function should look like this:

~~~xml
    <Function Name="CountCategories">
          <Parameter Name="Amount" Type="Edm.Int32" Nullable="false"/>
          <ReturnType Type="Collection(OData.Demo.Category)"/>
    </Function>
~~~

As described in the previous tutorials, the type of the response determines which processor interface will be called by the Olingo library. It is **important to know, that functions are dispatched to the traditional processor interfaces**.
That means there are no special "FunctionProcessors". In our case, the function returns a collection of Categories. So we have to extend the `DemoEntityCollectionProcessor`. As you will see it is possible to address a single entity by its  key. So we have to extend the `DemoEntityProcessor` as well to handle requests which responds a single entity.

**Action Import: Reset**    
This action takes an optional parameter “*Amount*”. The actions resets the whole data of the service and creates  *# of Amount* products with the related categories.

After finishing the implementation the definition of the action should be like this:

~~~xml
    <Action Name="Reset" IsBound="false">
      <Parameter Name="Amount" Type="Edm.Int32"/>
    </Action>
~~~

While actions are called by using HTTP Method POST is nessesary to introduce new processor interfaces for actions. So there exists a bunch of interfaces, for each return type strictly one.

**Steps**    

  * Extend the Metadata model
  * Extend the data store
  * Extend the entity collection and the entity processor to handle function imports
  * Implement an action processor

### Extend the Metadata model

Create the following constants in the DemoEdmProvider. These constants are used to address the operations.

~~~java
    // Action
    public static final String ACTION_RESET = "Reset";
    public static final FullQualifiedName ACTION_RESET_FQN = new FullQualifiedName(NAMESPACE, ACTION_RESET);

    // Function
    public static final String FUNCTION_COUNT_CATEGORIES = "CountCategories";
    public static final FullQualifiedName FUNCTION_COUNT_CATEGORIES_FQN = new FullQualifiedName(NAMESPACE, FUNCTION_COUNT_CATEGORIES);

    // Function/Action Parameters
    public static final String PARAMETER_AMOUNT = "Amount";
~~~

The way to announce the operations is very similar to announcing EntityTypes. We have to override some methods. Those methods provide the definition of the Edm elements. We need methods for:

  - Actions
  - Functions
  - Action Imports
  - Function Imports

The code is simple and straight forward. First, we check which function we have to return. Then, a list of parameters and the return type are created. At the end all parts are fit together and get returned as new `CsdlFunction` Object.

~~~java
    @Override
    public List<CsdlFunction> getFunctions(final FullQualifiedName functionName) {
      if (functionName.equals(FUNCTION_COUNT_CATEGORIES_FQN)) {
        // It is allowed to overload functions, so we have to provide a list of functions for each function name
        final List<CsdlFunction> functions = new ArrayList<CsdlFunction>();

        // Create the parameter for the function
        final CsdlParameter parameterAmount = new CsdlParameter();
        parameterAmount.setName(PARAMETER_AMOUNT);
        parameterAmount.setNullable(false);
        parameterAmount.setType(EdmPrimitiveTypeKind.Int32.getFullQualifiedName());

        // Create the return type of the function
        final CsdlReturnType returnType = new CsdlReturnType();
        returnType.setCollection(true);
        returnType.setType(ET_CATEGORY_FQN);

        // Create the function
        final CsdlFunction function = new CsdlFunction();
        function.setName(FUNCTION_COUNT_CATEGORIES_FQN.getName())
            .setParameters(Arrays.asList(parameterAmount))
            .setReturnType(returnType);
        functions.add(function);

        return functions;
      }

      return null;
    }
~~~

We have created the function itself. To express that function can be called statically we have to override the method `getFunctionImport()`.

~~~java
    @Override
    public CsdlFunctionImport getFunctionImport(FullQualifiedName entityContainer, String functionImportName) {
      if(entityContainer.equals(CONTAINER)) {
        if(functionImportName.equals(FUNCTION_COUNT_CATEGORIES_FQN.getName())) {
          return new CsdlFunctionImport()
                    .setName(functionImportName)
                    .setFunction(FUNCTION_COUNT_CATEGORIES_FQN)
                    .setEntitySet(ES_CATEGORIES_NAME)
                    .setIncludeInServiceDocument(true);
        }
      }

      return null;
    }
~~~

To define the actions and the action imports the `getActions()` and `getActionImport()` methods have to be overriden and the necessary code is quite similar to the functions sample above:

~~~java
    @Override
    public List<CsdlAction> getActions(final FullQualifiedName actionName) {
      if(actionName.equals(ACTION_RESET_FQN)) {
        // It is allowed to overload actions, so we have to provide a list of Actions for each action name
        final List<CsdlAction> actions = new ArrayList<CsdlAction>();

        // Create parameters
        final List<CsdlParameter> parameters = new ArrayList<CsdlParameter>();
        final CsdlParameter parameter = new CsdlParameter();
        parameter.setName(PARAMETER_AMOUNT);
        parameter.setType(EdmPrimitiveTypeKind.Int32.getFullQualifiedName());
        parameters.add(parameter);

        // Create the Csdl Action
        final CsdlAction action = new CsdlAction();
        action.setName(ACTION_RESET_FQN.getName());
        action.setParameters(parameters);
        actions.add(action);

        return actions;
      }

      return null;
    }

    @Override
    public CsdlActionImport getActionImport(final FullQualifiedName entityContainer, final String actionImportName) {
      if(entityContainer.equals(CONTAINER)) {
        if(actionImportName.equals(ACTION_RESET_FQN.getName())) {
          return new CsdlActionImport()
                  .setName(actionImportName)
                  .setAction(ACTION_RESET_FQN);
        }
      }

      return null;
    }
~~~

Finally we have to announce these operations to the schema and the entity container.
Add the following lines to the method `getSchemas()`:

~~~java
    // add actions
    List<CsdlAction> actions = new ArrayList<CsdlAction>();
    actions.addAll(getActions(ACTION_RESET_FQN));
    schema.setActions(actions);

    // add functions
    List<CsdlFunction> functions = new ArrayList<CsdlFunction>();
    functions.addAll(getFunctions(FUNCTION_COUNT_CATEGORIES_FQN));
    schema.setFunctions(functions);
~~~

Also add the following lines to the method `getEntityContainer()`

~~~java
    // Create function imports
    List<CsdlFunctionImport> functionImports = new ArrayList<CsdlFunctionImport>();
    functionImports.add(getFunctionImport(CONTAINER, FUNCTION_COUNT_CATEGORIES));

    // Create action imports
    List<CsdlActionImport> actionImports = new ArrayList<CsdlActionImport>();
    actionImports.add(getActionImport(CONTAINER, ACTION_RESET));

    entityContainer.setFunctionImports(functionImports);
    entityContainer.setActionImports(actionImports);
~~~

### Extend the data store

We need two methods in the data store to read the function import `CountCategories`.

The first method returns a collection of entites and the second returns a single entity of this collection.

~~~java
    public EntityCollection readFunctionImportCollection(final UriResourceFunction uriResourceFunction, final ServiceMetadata serviceMetadata) throws ODataApplicationException {

      if(DemoEdmProvider.FUNCTION_COUNT_CATEGORIES.equals(uriResourceFunction.getFunctionImport().getName())) {
        // Get the parameter of the function
        final UriParameter parameterAmount = uriResourceFunction.getParameters().get(0);

        // Try to convert the parameter to an Integer.
        // We have to take care, that the type of parameter fits to its EDM declaration
        int amount;
        try {
          amount = Integer.parseInt(parameterAmount.getText());
        } catch(NumberFormatException e) {
          throw new ODataApplicationException("Type of parameter Amount must be Edm.Int32",
            HttpStatusCode.BAD_REQUEST.getStatusCode(), Locale.ENGLISH);
        }

        final EdmEntityType productEntityType = serviceMetadata.getEdm().getEntityType(DemoEdmProvider.ET_PRODUCT_FQN);
        final List<Entity> resultEntityList = new ArrayList<Entity>();

        // Loop over all categories and check how many products are linked
        for(final Entity category : categoryList) {
          final EntityCollection products = getRelatedEntityCollection(category, productEntityType);
          if(products.getEntities().size() == amount) {
            resultEntityList.add(category);
          }
        }

        final EntityCollection resultCollection = new EntityCollection();
        resultCollection.getEntities().addAll(resultEntityList);
        return resultCollection;
      } else {
          throw new ODataApplicationException("Function not implemented", HttpStatusCode.NOT_IMPLEMENTED.getStatusCode(),
        Locale.ROOT);
      }
    }

    public Entity readFunctionImportEntity(final UriResourceFunction uriResourceFunction,
      final ServiceMetadata serviceMetadata) throws ODataApplicationException {

      final EntityCollection entityCollection = readFunctionImportCollection(uriResourceFunction, serviceMetadata);
      final EdmEntityType edmEntityType = (EdmEntityType) uriResourceFunction.getFunction().getReturnType().getType();

      return Util.findEntity(edmEntityType, entityCollection, uriResourceFunction.getKeyPredicates());
    }
~~~

We also create two methods to reset the data of our service.

~~~java
    public void resetDataSet(final int amount) {
      // Replace the old lists with empty ones
      productList = new ArrayList<Entity>();
      categoryList = new ArrayList<Entity>();

      // Create new sample data
      initProductSampleData();
      initCategorySampleData();

      // Truncate the lists
      if(amount < productList.size()) {
        productList = productList.subList(0, amount);
        // Products 0, 1 are linked to category 0
        // Products 2, 3 are linked to category 1
        // Products 4, 5 are linked to category 2
        categoryList = categoryList.subList(0, (amount / 2) + 1);
      }
    }

    public void resetDataSet() {
      resetDataSet(Integer.MAX_VALUE);
    }
~~~

### Extend the entity collection and the entity processor to handle function imports

We start with the entity collection processor `DemoEntityCollectionProcessor`.
To keep things simple, the first steps is to distinguish between entity collections and function imports.
A cleverer implementation can handle both cases in one method to avoid duplicated code.

The recent implementation of the `readEntityCollection()` has been moved to `readEntityCollectionInternal()`

~~~java
    public void readEntityCollection(ODataRequest request, ODataResponse response, UriInfo uriInfo, ContentType responseFormat) throws ODataApplicationException, SerializerException {

      final UriResource firstResourceSegment = uriInfo.getUriResourceParts().get(0);

      if(firstResourceSegment instanceof UriResourceEntitySet) {
        readEntityCollectionInternal(request, response, uriInfo, responseFormat);
      } else if(firstResourceSegment instanceof UriResourceFunction) {
        readFunctionImportCollection(request, response, uriInfo, responseFormat);
      } else {
        throw new ODataApplicationException("Not implemented",
          HttpStatusCode.NOT_IMPLEMENTED.getStatusCode(),
        Locale.ENGLISH);
      }
    }
~~~

Like by reading *entity collections*, the first step is to analyze the URI and then fetch the data (of the function import).

~~~java
    private void readFunctionImportCollection(final ODataRequest request, final ODataResponse response,
      final UriInfo uriInfo, final ContentType responseFormat) throws ODataApplicationException, SerializerException {

      // 1st step: Analyze the URI and fetch the entity collection returned by the function import
      // Function Imports are always the first segment of the resource path
      final UriResource firstSegment = uriInfo.getUriResourceParts().get(0);

      if(!(firstSegment instanceof UriResourceFunction)) {
        throw new ODataApplicationException("Not implemented",
          HttpStatusCode.NOT_IMPLEMENTED.getStatusCode(), Locale.ENGLISH);
      }

      final UriResourceFunction uriResourceFunction = (UriResourceFunction) firstSegment;
      final EntityCollection entityCol = storage.readFunctionImportCollection(uriResourceFunction, serviceMetadata);
~~~

Then the result has to be serialized. The only difference to entity sets is the way how the `EdmEntityType` is determined.

~~~java
      // 2nd step: Serialize the response entity
      final EdmEntityType edmEntityType = (EdmEntityType) uriResourceFunction.getFunction().getReturnType().getType();
      final ContextURL contextURL = ContextURL.with().asCollection().type(edmEntityType).build();
      EntityCollectionSerializerOptions opts = EntityCollectionSerializerOptions.with().contextURL(contextURL).build();
      final ODataSerializer serializer = odata.createSerializer(responseFormat);
      final SerializerResult serializerResult = serializer.entityCollection(serviceMetadata, edmEntityType, entityCol, opts);

      // 3rd configure the response object
      response.setContent(serializerResult.getContent());
      response.setStatusCode(HttpStatusCode.OK.getStatusCode());
      response.setHeader(HttpHeader.CONTENT_TYPE, responseFormat.toContentTypeString());
    }
~~~

Next we will implement the processor to read a *single entity*. The implementation is quite similar to the implementation of the collection processor.

~~~java
    public void readEntity(ODataRequest request, ODataResponse response, UriInfo uriInfo, ContentType responseFormat)
      throws ODataApplicationException, SerializerException {

      // The sample service supports only functions imports and entity sets.
      // We do not care about bound functions and composable functions.

      UriResource uriResource = uriInfo.getUriResourceParts().get(0);

      if(uriResource instanceof UriResourceEntitySet) {
        readEntityInternal(request, response, uriInfo, responseFormat);
      } else if(uriResource instanceof UriResourceFunction) {
        readFunctionImportInternal(request, response, uriInfo, responseFormat);
      } else {
        throw new ODataApplicationException("Only EntitySet is supported",
          HttpStatusCode.NOT_IMPLEMENTED.getStatusCode(), Locale.ENGLISH);
      }
    }

    private void readFunctionImportInternal(final ODataRequest request, final ODataResponse response,
      final UriInfo uriInfo, final ContentType responseFormat) throws ODataApplicationException, SerializerException {

      // 1st step: Analyze the URI and fetch the entity returned by the function import
      // Function Imports are always the first segment of the resource path
      final UriResource firstSegment = uriInfo.getUriResourceParts().get(0);

      if(!(firstSegment instanceof UriResourceFunction)) {
        throw new ODataApplicationException("Not implemented",
          HttpStatusCode.NOT_IMPLEMENTED.getStatusCode(), Locale.ENGLISH);
      }

      final UriResourceFunction uriResourceFunction = (UriResourceFunction) firstSegment;
      final Entity entity = storage.readFunctionImportEntity(uriResourceFunction, serviceMetadata);

      if(entity == null) {
        throw new ODataApplicationException("Nothing found.",
          HttpStatusCode.NOT_FOUND.getStatusCode(), Locale.ROOT);
      }

      // 2nd step: Serialize the response entity
      final EdmEntityType edmEntityType = (EdmEntityType) uriResourceFunction.getFunction().getReturnType().getType();
      final ContextURL contextURL = ContextURL.with().type(edmEntityType).build();
      final EntitySerializerOptions opts = EntitySerializerOptions.with().contextURL(contextURL).build();
      final ODataSerializer serializer = odata.createSerializer(responseFormat);
      final SerializerResult serializerResult = serializer.entity(serviceMetadata, edmEntityType, entity, opts);

      // 3rd configure the response object
      response.setContent(serializerResult.getContent());
      response.setStatusCode(HttpStatusCode.OK.getStatusCode());
      response.setHeader(HttpHeader.CONTENT_TYPE, responseFormat.toContentTypeString());
    }
~~~

### Implement an action processor

Create a new class `DemoActionProcessor` make them implement the interface `ActionVoidProcessor`.

~~~java
    public class DemoActionProcessor implements ActionVoidProcessor {

      private OData odata;
      private Storage storage;

      public DemoActionProcessor(final Storage storage) {
        this.storage = storage;
      }

      @Override
      public void init(final OData odata, final ServiceMetadata serviceMetadata)   {
        this.odata = odata;
      }
~~~

First analyze the uri.

~~~java
    public void processActionVoid(ODataRequest request, ODataResponse response, UriInfo uriInfo,
        ContentType requestFormat) throws ODataApplicationException, ODataLibraryException {

      // 1st Get the action from the resource path
      final EdmAction edmAction = ((UriResourceAction) uriInfo.asUriInfoResource().getUriResourceParts()
                                            .get(0)).getAction();
~~~

Then deserialize the *action parameters*.

~~~java
      // 2nd Deserialize the parameter
      // In our case there is only one action. So we can be sure that parameter "Amount" has been provided by the client
      if (requestFormat == null) {
        throw new ODataApplicationException("The content type has not been set in the request.",
          HttpStatusCode.BAD_REQUEST.getStatusCode(), Locale.ROOT);
      }

      final ODataDeserializer deserializer = odata.createDeserializer(requestFormat);
      final Map<String, Parameter> actionParameter = deserializer.actionParameters(request.getBody(), edmAction)
                                     .getActionParameters();
      final Parameter parameterAmount = actionParameter.get(DemoEdmProvider.PARAMETER_AMOUNT);
~~~

Execute the action and set the response code.

~~~java
      // The parameter amount is nullable
      if(parameterAmount.isNull()) {
        storage.resetDataSet();
      } else {
        final Integer amount = (Integer) parameterAmount.asPrimitive();
        storage.resetDataSet(amount);
      }

      response.setStatusCode(HttpStatusCode.NO_CONTENT.getStatusCode());
    }
~~~

## Run the implemented service

After building and deploying your service to your server, you can try the following requests:

**Functions (Called via GET)**    

  * [http://localhost:8080/DemoService-Action/DemoService.svc/CountCategories(Amount=2)](http://localhost:8080/DemoService-Action/DemoService.svc/CountCategories(Amount=2))
  * [http://localhost:8080/DemoService-Action/DemoService.svc/CountCategories(Amount=2)(0)](http://localhost:8080/DemoService-Action/DemoService.svc/CountCategories(Amount=2)(0))

**Actions (Called via POST)**     
*Note:* Set the Content-Type header to: `Content-Type: application/json`

  * [http://localhost:8080/DemoService-Action/DemoService.svc/Reset](http://localhost:8080/DemoService-Action/DemoService.svc/Reset)

    Content:

    { }


  * [http://localhost:8080/DemoService-Action/DemoService.svc/Reset](http://localhost:8080/DemoService-Action/DemoService.svc/Reset)

    Content:

    { "Amount": 1 }


To verify that the service has been reseted, you can request the collection of products

* [http://localhost:8080/DemoService-Action/DemoService.svc/Products](http://localhost:8080/DemoService-Action/DemoService.svc/Products)
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
