Title: Read Scenario

# Read Scenario - Read with $expand

---

### How To Guide - Extend basic read scenario with support for $expand

This How To Guide shows how to extend the basic read scenario with support for the $expand system query option.
It shows how to call the `EntityProvider.writeEntry(...)` and `EntityProvider.writeEntrySet(...)` methods with the necessary `EntityProviderWriteProperties` set and how to implement the necessary `OnWriteEntryContent OnWriteFeedContent` callbacks.

### Prerequisites

Setup of [Basic Read Scenario](basicread)

### Shortcut 

If you like to directly experiment with the results of the extented basic read scenario, you can use this shortcut: 

  - Download and unzip [Olingo Tutorial 'Basic Read with $expand extension' Project](apache-olingo-tutorial-adv_read_expand.zip) to your local drive which is your OData Tutorial project folder (referenced as `$ODATA_PROJECT_HOME` in the tutorial).
  - Start the command line tool and execute the following command in the folder `$ODATA_PROJECT_HOME` 
    - `mvn eclipse:eclipse clean install` 
  - Go into Eclipse and import the project into your workspace by... 
    - Menu *File -> Import*... 
    - *Existing projects into workspace*, then choose the `$ODATA_PROJECT_HOME` folder 
    - Select both projects *olingo.odata2.sample.service* and *olingo.odata2.sample.web* and press *Finish*. 

### Set Up your development project 

If [Basic Read Scenario](basicread) is already set up there is nothing additional to do. Otherwise please refer to the Prerequisites section of the [Basic Read Scenario](basicread).

### Extend Basic Read Scenario
The steps to extend the basic read with $expand support for the Car and Manufacturer entities (not entity sets) are to provide the expanded data via ODataCallbacks and register these for the corresponding navigation properties. 

### Implement OnWriteEntryContent and OnWriteFeedContent callbacks
To support `$expand` for a single entry the interface `org.apache.olingo.odata2.api.ep.callback.OnWriteEntryContent` must be implemented. This provides the method `WriteEntryCallbackResult retrieveEntryResult(WriteEntryCallbackContext context) throws ODataApplicationException;` which is called during processing from the `EntityProvider` to receive the necessary data which than is inlined in the response.

In our sample we create a class `MyCallback` which implements `org.apache.olingo.odata2.api.ep.callback.OnWriteEntryContent` in following way:

##### Sample Code

```java
@Override
public WriteEntryCallbackResult retrieveEntryResult(WriteEntryCallbackContext context) throws ODataApplicationException {
WriteEntryCallbackResult result = new WriteEntryCallbackResult();

  try {
    if (isNavigationFromTo(context, ENTITY_SET_NAME_CARS, ENTITY_NAME_MANUFACTURER)) {
    EntityProviderWriteProperties inlineProperties = EntityProviderWriteProperties.serviceRoot(serviceRoot)
        .expandSelectTree(context.getCurrentExpandSelectTreeNode())
        .build();

      Map<String, Object> keys = context.extractKeyFromEntryData();
      Integer carId = (Integer) keys.get("Id");
      result.setEntryData(dataStore.getManufacturerFor(carId));
      result.setInlineProperties(inlineProperties);
    }
  } catch (EdmException e) {
  // TODO: should be handled and not only logged
  LOG.error("Error in $expand handling.", e);
  } catch (EntityProviderException e) {
  // TODO: should be handled and not only logged
  LOG.error("Error in $expand handling.", e);
  }
    
  return result;
}
```

Within this method we first check if the source entity and navigation property are correct for our case (via the method `isNavigationFromTo(...):boolean)`, then we create the `EntityProviderWriteProperties` with the new (current) `ExpandSelectTreeNode`, receive the data from our `DataStore` and put all into the result which then will be further processed by the `EntityProvider`.

### Implementation for $expand for an entity set
To support `$expand` for a feed of entries (entity set) the interface `org.apache.olingo.odata2.api.ep.callback.OnWriteFeedContent` must be implemented. These provides the method `WriteFeedCallbackResult retrieveFeedResult(WriteFeedCallbackContext context) throws ODataApplicationException;` which is called during processing from the `EntityProvider` to receive the necessary data which than is inlined in the response.

It is possible to create an additional callback class but for convenience we expand our already created callback (`MyCallback`) to implement `org.apache.olingo.odata2.api.ep.callback.OnWriteFeedContent` and provide the method implementation in following way:

##### Sample Code

```java
@Override
public WriteFeedCallbackResult retrieveFeedResult(WriteFeedCallbackContext context) throws ODataApplicationException {
WriteFeedCallbackResult result = new WriteFeedCallbackResult();
  try {
    if(isNavigationFromTo(context, ENTITY_SET_NAME_MANUFACTURERS, ENTITY_SET_NAME_CARS)) {
    EntityProviderWriteProperties inlineProperties = EntityProviderWriteProperties.serviceRoot(serviceRoot)
        .expandSelectTree(context.getCurrentExpandSelectTreeNode())
        .selfLink(context.getSelfLink())
        .build();

      Map<String, Object> keys = context.extractKeyFromEntryData();
      Integer manufacturerId = (Integer) keys.get("Id");
      result.setFeedData(dataStore.getCarsFor(manufacturerId));
      result.setInlineProperties(inlineProperties);
    }
  } catch (EdmException e) {
  // TODO: should be handled and not only logged
  LOG.error("Error in $expand handling.", e);
  } catch (EntityProviderException e) {
  // TODO: should be handled and not only logged
  LOG.error("Error in $expand handling.", e);
  }
  return result;
}
```

Within this method we first check if the source entity and navigation property are correct for our case (via the method `isNavigationFromTo(...):boolean)`, then we create the `EntityProviderWriteProperties` with the new (current) `ExpandSelectTreeNode`, receive the data from our `DataStore` and put all into the result which then will be further processed by the `EntityProvider`.

This example shows that the basic callback logic between `OnWriteEntryConten`t and `OnWriteFeedContent` is very similar. Validation of current element (optional), preparing of `EntityProviderWriteProperties`, receive of data and putting all together into corresponding result object (`WriteEntryCallbackResult` or `WriteFeedCallbackResult`).

To improve code readability the `isNavigationFromTo(...):boolean` method was also added to the class. The method is used to check if the retrieved request is related to given entity set and navigation:

#### Sample Code

```java
private boolean isNavigationFromTo(WriteCallbackContext context, String entitySetName, String navigationPropertyName) throws EdmException {
  if(entitySetName == null || navigationPropertyName == null) {
  return false;
  }
  EdmEntitySet sourceEntitySet = context.getSourceEntitySet();
  EdmNavigationProperty navigationProperty = context.getNavigationProperty();
  return entitySetName.equals(sourceEntitySet.getName()) && navigationPropertyName.equals(navigationProperty.getName());
}
```

### Extend ODataSingleProcessor.readEntity(...)
The necessary callbacks (`MyCallback` class) now has to be registered during the corresponding `readEntity(...)` call. Therefore we first create a map with the property name as key and the according callback as value. Additional we need to create the `ExpandSelectTreeNode` based on current element position. Both then have to be set in the `EntityProviderWritePropertiesBuilder`. 

The following code show the few lines we need for extending the read of a car with its expanded manufacturer.

```java
// create and register callback
Map<String, ODataCallback> callbacks = new HashMap<String, ODataCallback>();
callbacks.put(ENTITY_NAME_MANUFACTURER, new MyCallback(dataStore, serviceRoot));
ExpandSelectTreeNode expandSelectTreeNode = UriParser.createExpandSelectTree(uriInfo.getSelect(), uriInfo.getExpand());
propertiesBuilder.expandSelectTree(expandSelectTreeNode).callbacks(callbacks);
```

The following code show the few lines we need for extending the read of a manufacturer with its expanded cars.

```java
// create and register callback
Map<String, ODataCallback> callbacks = new HashMap<String, ODataCallback>();
callbacks.put(ENTITY_SET_NAME_CARS, new MyCallback(dataStore, serviceRoot));
ExpandSelectTreeNode expandSelectTreeNode = UriParser.createExpandSelectTree(uriInfo.getSelect(), uriInfo.getExpand());
propertiesBuilder.expandSelectTree(expandSelectTreeNode).callbacks(callbacks);
```

The complete `readEntity(...)` method should now look like:

```java
public ODataResponse readEntity(GetEntityUriInfo uriInfo, String contentType) throws ODataException {
    
  if (uriInfo.getNavigationSegments().size() == 0) {
  EdmEntitySet entitySet = uriInfo.getStartEntitySet();

    if (ENTITY_SET_NAME_CARS.equals(entitySet.getName())) {
    int id = getKeyValue(uriInfo.getKeyPredicates().get(0));
    Map<String, Object> data = dataStore.getCar(id);
        
      if (data != null) {
        URI serviceRoot = getContext().getPathInfo().getServiceRoot();
        ODataEntityProviderPropertiesBuilder propertiesBuilder = EntityProviderWriteProperties.serviceRoot(serviceRoot);
          
        // create and register callback
        Map<String, ODataCallback> callbacks = new HashMap<String, ODataCallback>();
        callbacks.put(ENTITY_NAME_MANUFACTURER, new MyCallback(dataStore, serviceRoot));
        ExpandSelectTreeNode expandSelectTreeNode = UriParser.createExpandSelectTree(uriInfo.getSelect(), uriInfo.getExpand());
        //
        propertiesBuilder.expandSelectTree(expandSelectTreeNode).callbacks(callbacks);

        return EntityProvider.writeEntry(contentType, entitySet, data, propertiesBuilder.build());
      }
    } else if (ENTITY_SET_NAME_MANUFACTURERS.equals(entitySet.getName())) {
      int id = getKeyValue(uriInfo.getKeyPredicates().get(0));
      Map<String, Object> data = dataStore.getManufacturer(id);
        
      if (data != null) {
        URI serviceRoot = getContext().getPathInfo().getServiceRoot();
        ODataEntityProviderPropertiesBuilder propertiesBuilder = EntityProviderWriteProperties.serviceRoot(serviceRoot);
  
        // create and register callback
        Map<String, ODataCallback> callbacks = new HashMap<String, ODataCallback>();
        callbacks.put(ENTITY_SET_NAME_CARS, new MyCallback(dataStore, serviceRoot));
        ExpandSelectTreeNode expandSelectTreeNode = UriParser.createExpandSelectTree(uriInfo.getSelect(), uriInfo.getExpand());
        //
        propertiesBuilder.expandSelectTree(expandSelectTreeNode).callbacks(callbacks);

        return EntityProvider.writeEntry(contentType, entitySet, data, propertiesBuilder.build());
      }
    }

    throw new ODataNotFoundException(ODataNotFoundException.ENTITY);

  } else if (uriInfo.getNavigationSegments().size() == 1) {
    //navigation first level, simplified example for illustration purposes only
    EdmEntitySet entitySet = uriInfo.getTargetEntitySet();
    if (ENTITY_SET_NAME_MANUFACTURERS.equals(entitySet.getName())) {
      int carKey = getKeyValue(uriInfo.getKeyPredicates().get(0));
      return EntityProvider.writeEntry(contentType, uriInfo.getTargetEntitySet(), dataStore.getManufacturer(carKey),   EntityProviderWriteProperties.serviceRoot(getContext().getPathInfo().getServiceRoot()).build());
    }

    throw new ODataNotFoundException(ODataNotFoundException.ENTITY);
  }

  throw new ODataNotImplementedException();
}
```

Now we can test out `$expand` extension in the web application.

### Deploy, run and test $expand

Like in the basic read scenario follow these steps:

  - Build your project: `mvn clean install` 
  - When build finished in Eclipse, run the Web Application via *Run As -> Run on Server* 
  - After successful server start and deployment the following uris from the basic read sample work as before: 
    - Show the Manufacturers: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers) 
    - Show one Manufacturer: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)) 
    - Show the Cars: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars) 
    - Show one Car: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)) 
    - Show the related Manufacturer of a Car: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)/Manufacturer](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)/Manufacturer) 
    - Show the related Cars of a Manufacturer: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)/Cars](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)/Cars) 
  - And in addition we can now expand the car and manufacturer with each other: 
    - Show Car with its Manufacturer: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)?$expand=Manufacturer ](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)?$expand=Manufacturer )
    - Show Manufacturer with its Cars: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)?$expand=Cars](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)?$expand=Cars) 

### Further Information

Next extension step for read scenario are read of [Media Resources](read_media-resource). 

  