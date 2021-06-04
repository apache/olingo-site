Title: Media Resources

# Media Resources 

---

### How To Guide - Extend read scenario with support for media resources

This How To Guide shows how to extend the read scenario (with `$expand` extension) with support for Media Link Entries and Media Resources.

The tutorial introduces a new resource (Driver), shows how to extend the according `readEntitySet(...)` and `readEntity(...)` and introduces the new method `readEntityMedia(uriInfo:GetMediaResourceUriInfo, contentType:String):ODataResponse` within the `MyODataSingleProcessor`.

### Prerequisites
Setup of [Read Scenario with `$expand`](read_expand) extension
(which has as prerequisite Setup of [Basic Read Scenario](basicread))


### Shortcut
If you like to directly experiment with the results of the extented basic read scenario, you can use this shortcut: 

  - Download and unzip the [Olingo Tutorial 'Basic Read with Media Resource extension' Project](apache-olingo-tutorial-adv_read_mediaresource.zip) to your local drive which is your OData Tutorial project folder (referenced as `$ODATA_PROJECT_HOME` in the turorial).
  - Start the command line tool and run maven in the folder `$ODATA_PROJECT_HOME` to build the war file which then can be deployed. 
  - Deploy the resulting war to your favorite application server. 
    - For a Tomcat application server copy the war into the *$TOMCAT_HOME/webapps* folder and start the server via `$TOMCAT_HOME/bin/startup.sh` (at Windows via `$TOMCAT_HOME/bin/startup.bat`). 
  - Optional: To import the sample project into Eclipse run the following steps:
    - Start the command line tool and run `mvn eclipse:eclipse clean install` in the folder `$ODATA_PROJECT_HOME` to generate the Eclipse project settings and do an initial build. 
    - Go into Eclipse and import the project into your workspace by...
      - Menue "File" -> "Import..."
      - "Existing projects into workspace" then choose the olingo.odata2.sample folder
      - Select both projects olingo.odata2.sample.service and olingo.odata2.sample.web and press "Finish"

### Hints
  - Sometimes a code extension needs one or more new/additional imports. These are not shown in the examples because it would blow up the source code examples and normally the IDE should support an auto fix of missing imports during typing of the code or after an insert of copied code. 

### Extend Read Scenario
The steps to extend the read scenario (already with `$expand` support) with support for a Driver which is a media link entry with an associated media resource and therefore contains an image as media is to override and implement the `readEntityMedia(uriInfo:GetMediaResourceUriInfo, contentType:String):ODataResponse` method within the `MyODataSingleProcessor`.

All $value requests - where $value is the next segment to an entity type instance identified by an EntitySet with key predicate or a navigation property (to one relation or key predicate for to many relation) - to the service are delegated to this method which then handles the creation of the according response. 

### Extend MyEdmProvider and MyODataSingleProcessor

#### Small sample code Prerequisites
Not directly related to the OData Library parts but necessary for the sample project is the extension of the DataStore with methods to access the new Driver and the binary data for the media resource of the Driver.
Because this extension is more boilerplate code with no direct OData impact it is recommended to simply copy the DataStore.java into the project.
As conclusion following methods are added which are accessed from the `MyODataSingleProcessor`:

  - `readDriverImage(entitySet:EdmEntitySet, id:int):byte[]` 
  - `getDriverFor(carId:int):Map<String, Object> `
  - `getCarFor(driverKey:int):Map<String, Object>` 
  - `getDriver(id:int):Map<String, Object> `
  - `getDrivers():List<Map<String, Object>>` 

Additional and in conclusion with the `DataStore` three images (png) are added to the sample project. These images are located in */sample-service/src/main/resources/* and are named Driver_'x'.png were 'x' is replaced by a number.
To really get an image for a media resource request it is necessary to copy these images (or create own PNG files which follows the naming schema).
If nothing is copied/created the sample will still work but does not show an image for a media resource request (instead a HTTP Response 404 - Entity not found is shown). 

** Extend MyEdmProvider with Driver data model **<br>

First the new Driver is added in the EDM. 
The Driver gets a Key (Id) and some properties (Name, Surname, Nickname, Updates). For simplification currently we do not add any associations to another entities.

To mark the Driver as a media link entry with an associated media resource extension it is mandatory to set the stream property to true via the `setHasStream(true)` method when creating the `EntityType`.

As conclusion the `getEntityType(…)` method is extended as follows:

##### Sample Code

```java
…
  } else if (ENTITY_TYPE_1_3.getName().equals(edmFQName.getName())) {
    List<Property> properties = new ArrayList<Property>();
    properties.add(new SimpleProperty().setName("Id").setType(EdmSimpleTypeKind.Int32).setFacets(new Facets().setNullable(false)));
    properties.add(new SimpleProperty().setName("Name").setType(EdmSimpleTypeKind.String).setFacets(new Facets().setNullable(false).setMaxLength(50)));
    properties.add(new SimpleProperty().setName("Surname").setType(EdmSimpleTypeKind.String).setFacets(new Facets().setNullable(false).setMaxLength(80))
                .setCustomizableFeedMappings(new CustomizableFeedMappings().setFcTargetPath(EdmTargetPath.SYNDICATION_TITLE)));
    properties.add(new SimpleProperty().setName("Nickname").setType(EdmSimpleTypeKind.String).setFacets(new Facets().setNullable(true).setMaxLength(50)));
    properties.add(new SimpleProperty().setName("Updated").setType(EdmSimpleTypeKind.DateTime)
        .setFacets(new Facets().setNullable(false).setConcurrencyMode(EdmConcurrencyMode.Fixed))
        .setCustomizableFeedMappings(new CustomizableFeedMappings().setFcTargetPath(EdmTargetPath.SYNDICATION_UPDATED)));

    // Navigation properties
    List<NavigationProperty> navigationProperties = new ArrayList<NavigationProperty>();
        
    // Key
    List<PropertyRef> keyProperties = new ArrayList<PropertyRef>();
    keyProperties.add(new PropertyRef().setName("Id"));
    Key key = new Key().setKeys(keyProperties);

    // finish
    return new EntityType().setName(ENTITY_TYPE_1_3.getName())
        .setProperties(properties)
        .setHasStream(true)
        .setKey(key)
        .setNavigationProperties(navigationProperties)
        .setMapping(new Mapping().setMimeType("image/png"));
}
...
```

In addition it is necessary to extend the `getSchemas(…)` method with the according `EntityType` and `EntitySet` of the new Driver.

```java
entityTypes.add(getEntityType(ENTITY_TYPE_1_3));
entitySets.add(getEntitySet(ENTITY_CONTAINER, ENTITY_SET_NAME_DRIVERS));
```

And at last following constants are added to the `MyEdmProvider` for cleaner code.

```java
  static final String ENTITY_SET_NAME_DRIVERS = "Drivers";
  static final String ENTITY_NAME_DRIVER = "Driver";
  private static final FullQualifiedName ENTITY_TYPE_1_3 = new FullQualifiedName(NAMESPACE, ENTITY_NAME_DRIVER);
```

**Extend `MyODataSingleProcessor` with `readEntityMedia(uriInfo:GetMediaResourceUriInfo, contentType:String):ODataResponse` method**<br>
All requests for media resources are done via the specified $value property in the URL (e.g. *.../MyODataSample.svc/Drivers(1)/$value*). Such a request will be dispatched to an `EntityMediaProcessor` which is in our case the `MyODataSingleProcessor` (inherited from `ODataSingleProcessor`).
To handle now such read requests for our media resources we override and implement the `readEntityMedia(uriInfo:GetMediaResourceUriInfo, contentType:String):ODataResponse` method.
For our scenario we simply have to validate the correct requested target `EntitySet`, get the key for requesting the data from our `DataStore` (which contains the binary data of our media resource), use the `EntityProvider` to write the data and at last build the `ODataResponse`.
Seems a lot but in code this are only these few lines:


```java
@Override
public ODataResponse readEntityMedia(final GetMediaResourceUriInfo uriInfo, final String contentType) throws ODataException {
    
  final EdmEntitySet entitySet = uriInfo.getTargetEntitySet();
  if(ENTITY_SET_NAME_DRIVERS.equals(entitySet.getName())) {
    int id = getKeyValue(uriInfo.getKeyPredicates().get(0));
    byte[] image = dataStore.readDriverImage(entitySet, id);
    if (image == null) {
      throw new ODataNotFoundException(ODataNotFoundException.ENTITY);
    }
  
    String mimeType = "image/png";
    return ODataResponse.fromResponse(EntityProvider.writeBinary(mimeType, image)).build();
  }
    
  throw new ODataNotImplementedException();
}
```

With these extension it is possible to read the media resource, but for access to the Driver `EntitySet` and `Entity` the according read methods have to be extended.

** Extend existing `readEntitySet(...)` and `readEntity(...)` methods in `MyODataSingleProcessor` methods **<br>
For access to the Driver as `Entity` and as `EntitySet` the according `readreadEntitySet(...)` and `readEntity(...)` methods have to be extended.

But its quite the same procedure as in the basic read. Validate the requested Entity, get the key for requesting the `DataStore` and write the result data via the `EntityProvider`.

The resulting extension for `readEntity(…)`:

```java
  } else if (ENTITY_SET_NAME_DRIVERS.equals(entitySet.getName())) {
    int id = getKeyValue(uriInfo.getKeyPredicates().get(0));
    Map<String, Object> data = dataStore.getDriver(id);

    if (data != null) {
      URI serviceRoot = getContext().getPathInfo().getServiceRoot();
      ODataEntityProviderPropertiesBuilder propertiesBuilder = EntityProviderWriteProperties.serviceRoot(serviceRoot);

      return EntityProvider.writeEntry(contentType, entitySet, data, propertiesBuilder.build());
    }
```

and `readEntitySet(…)`:

```java
  } else if (ENTITY_SET_NAME_DRIVERS.equals(entitySet.getName())) {
    return EntityProvider.writeFeed(contentType, entitySet, dataStore.getDrivers(),     EntityProviderWriteProperties.serviceRoot(getContext().getPathInfo().getServiceRoot()).build());
```

**Conclusion for media resource extension**<br>
After finishing all steps above the project can be built and deployed containing a Driver type which is a media link entry with associated media resource. *Congratulations.*

For a more interesting sample and to update before learned knowledge about associations and `$expand` it is recommended to finish this HowTo with the creation of an association between a Driver and his Car with supported `$expand`.

**Extend Driver with association to Car**<br>
For a more interesting sample we now create an association between a Driver and his Car.

**Extend Driver and Car in MyEdmProvider with a navigation property**<br>
At first we introduce the necessary constants:

```java
  private static final FullQualifiedName ASSOCIATION_DRIVER_CAR = new FullQualifiedName(NAMESPACE, "Driver_Car-Car_Driver");
    
  private static final String ROLE_1_3 = "Car_Driver";
  private static final String ROLE_3_1 = "Driver_Car";
    
  private static final String ASSOCIATION_SET = "Cars_Manufacturers";
  private static final String ASSOCIATION_SET_CARS_DRIVERS = "Cars_Drivers";
```

Then the `getSchemas()` in `MyEdmProvider` is extended with the new association:

```java
associations.add(getAssociation(ASSOCIATION_DRIVER_CAR));
...
associationSets.add(getAssociationSet(ENTITY_CONTAINER, ASSOCIATION_DRIVER_CAR, ENTITY_SET_NAME_DRIVERS, ROLE_3_1));
```

Next step is the extension of the entity types in `getEntityType()` in `MyEdmProvider`.

For the Car:

```java
  if (ENTITY_TYPE_1_1.getName().equals(edmFQName.getName())) {
...
    navigationProperties.add(new NavigationProperty().setName("Driver")
        .setRelationship(ASSOCIATION_DRIVER_CAR).setFromRole(ROLE_1_3).setToRole(ROLE_3_1));
…
}
```

And the Driver:

```java
  if (ENTITY_TYPE_1_3.getName().equals(edmFQName.getName())) {
...
    properties.add(new SimpleProperty().setName("CarId").setType(EdmSimpleTypeKind.Int32));
...
    navigationProperties.add(new NavigationProperty().setName("Car").setRelationship(ASSOCIATION_DRIVER_CAR).setFromRole(ROLE_3_1).setToRole(ROLE_1_3));
...
}
```

At last the `getAssociation(…)` and `getAssociationSet(...)` has also to be extended and has to look like:

```java
@Override
public Association getAssociation(FullQualifiedName edmFQName) throws ODataException {
  if (NAMESPACE.equals(edmFQName.getNamespace())) {
    if (ASSOCIATION_CAR_MANUFACTURER.getName().equals(edmFQName.getName())) {
      return new Association().setName(ASSOCIATION_CAR_MANUFACTURER.getName())
        .setEnd1(new AssociationEnd().setType(ENTITY_TYPE_1_1).setRole(ROLE_1_1).setMultiplicity(EdmMultiplicity.MANY))
        .setEnd2(new AssociationEnd().setType(ENTITY_TYPE_1_2).setRole(ROLE_1_2).setMultiplicity(EdmMultiplicity.ONE));
    } else if (ASSOCIATION_DRIVER_CAR.getName().equals(edmFQName.getName())) {
      return new Association().setName(ASSOCIATION_DRIVER_CAR.getName())
        .setEnd1(new AssociationEnd().setType(ENTITY_TYPE_1_1).setRole(ROLE_1_3).setMultiplicity(EdmMultiplicity.ONE))
        .setEnd2(new AssociationEnd().setType(ENTITY_TYPE_1_3).setRole(ROLE_3_1).setMultiplicity(EdmMultiplicity.ONE));
    }
  }
  return null;
}

@Override
public AssociationSet getAssociationSet(String entityContainer, FullQualifiedName association, String sourceEntitySetName, String sourceEntitySetRole) throws ODataException {
  if (ENTITY_CONTAINER.equals(entityContainer)) {
    if (ASSOCIATION_CAR_MANUFACTURER.equals(association)) {
      return new AssociationSet().setName(ASSOCIATION_SET)
        .setAssociation(ASSOCIATION_CAR_MANUFACTURER)
        .setEnd1(new AssociationSetEnd().setRole(ROLE_1_2).setEntitySet(ENTITY_SET_NAME_MANUFACTURERS))
        .setEnd2(new AssociationSetEnd().setRole(ROLE_1_1).setEntitySet(ENTITY_SET_NAME_CARS));
    } else if (ASSOCIATION_DRIVER_CAR.equals(association)) {
      return new AssociationSet().setName(ASSOCIATION_SET_CARS_DRIVERS)
        .setAssociation(ASSOCIATION_DRIVER_CAR)
        .setEnd1(new AssociationSetEnd().setRole(ROLE_3_1).setEntitySet(ENTITY_SET_NAME_DRIVERS))
        .setEnd2(new AssociationSetEnd().setRole(ROLE_1_3).setEntitySet(ENTITY_SET_NAME_CARS));
    }
  }
  return null;
}
```

**Extend existing `readreadEntitySet(...)` and `readEntity(...)` methods in `MyODataSingleProcessor`**<br>
For cleaner code we introduce at first following method in the `MyODataSingleProcessor` which validate if the uri contains the expected association.

```java
private boolean isAssociation(GetEntityUriInfo uriInfo, String startName, String targetName) throws EdmException {
  if(startName == null || targetName == null) {
    return false;
  }
  EdmEntitySet startEntitySet = uriInfo.getStartEntitySet();
  EdmEntitySet targetEntitySet = uriInfo.getTargetEntitySet();
    
  return startName.equals(startEntitySet.getName()) && targetName.equals(targetEntitySet.getName());
}
```

The procedure should be already familiar. At first it is checked for the correct association of the requested Entity, then the key for requesting the DataStore is get as well as the data and then result data is written via the `EntityProvider`.

```java
} else if (uriInfo.getNavigationSegments().size() == 1) {
  //navigation first level, simplified example for illustration purposes only
  EdmEntitySet entitySet = uriInfo.getTargetEntitySet();
      
  Map<String, Object> data = null;
      
  if (ENTITY_SET_NAME_MANUFACTURERS.equals(entitySet.getName())) {
    int carKey = getKeyValue(uriInfo.getKeyPredicates().get(0));
    data = dataStore.getManufacturerFor(carKey);
  } else if(isAssociation(uriInfo, ENTITY_SET_NAME_CARS, ENTITY_SET_NAME_DRIVERS)) {
    int carKey = getKeyValue(uriInfo.getKeyPredicates().get(0));
    data = dataStore.getDriverFor(carKey);
  } else if(isAssociation(uriInfo, ENTITY_SET_NAME_DRIVERS, ENTITY_SET_NAME_CARS)) {
    int driverKey = getKeyValue(uriInfo.getKeyPredicates().get(0));
    data = dataStore.getCarFor(driverKey);
  }
      
  if(data != null) {
    return EntityProvider.writeEntry(contentType, uriInfo.getTargetEntitySet(), 
        data, EntityProviderWriteProperties.serviceRoot(getContext().getPathInfo().getServiceRoot()).build());
  }

  throw new ODataNotFoundException(ODataNotFoundException.ENTITY);
}
```

**Add $expand support for Driver to/from Car association**<br>
The last missing step is to add the `$expand` support for the new Driver to/from Car association.

**Extend MyCallback for Driver and Car association**<br>
Add first the extension in the MyCallback for each entity is done in the `retrieveEntryResult()` method.
The procedure is similar to the Cars - Manufacturers association. At first it is checked for the correct association of the requested Entity, then the key for requesting the DataStore is get as well as the data and then result data is attached to the `WriteEntryCallbackResult`.

The resulting method extension is:

```java
...
  if(isNavigationFromTo(context, ENTITY_SET_NAME_CARS, ENTITY_NAME_DRIVER)) {
    EntityProviderWriteProperties inlineProperties = EntityProviderWriteProperties.serviceRoot(serviceRoot)
        .expandSelectTree(context.getCurrentExpandSelectTreeNode())
        .build();

    Map<String, Object> keys = context.extractKeyFromEntryData();
    Integer carId = (Integer) keys.get("Id");
    result.setEntryData(dataStore.getDriverFor(carId));
    result.setInlineProperties(inlineProperties);
        
  } else if(isNavigationFromTo(context, ENTITY_SET_NAME_DRIVERS, ENTITY_NAME_CAR)) {
    EntityProviderWriteProperties inlineProperties = EntityProviderWriteProperties.serviceRoot(serviceRoot)
        .expandSelectTree(context.getCurrentExpandSelectTreeNode())
        .build();

    Map<String, Object> keys = context.extractKeyFromEntryData();
    Integer driverId = (Integer) keys.get("Id");
    result.setEntryData(dataStore.getCarFor(driverId));
    result.setInlineProperties(inlineProperties);        
  }
...
```

**Add registration of MyCallback for Driver and Car association**<br>
After extension of `MyCallback` it is necessary to register a callback within the `readEntity()` in the `MyODataSingleProcessor`.

For the Driver we add the complete callback registration (code between the comments) which results in final code for the complete Driver Entity handling:

```java
...
   } else if (ENTITY_SET_NAME_DRIVERS.equals(entitySet.getName())) {
    int id = getKeyValue(uriInfo.getKeyPredicates().get(0));
    Map<String, Object> data = dataStore.getDriver(id);

    if (data != null) {
      URI serviceRoot = getContext().getPathInfo().getServiceRoot();
      ODataEntityProviderPropertiesBuilder propertiesBuilder = EntityProviderWriteProperties.serviceRoot(serviceRoot);

      // create and register callback
      Map<String, ODataCallback> callbacks = new HashMap<String, ODataCallback>();
      callbacks.put(ENTITY_NAME_CAR, new MyCallback(dataStore, serviceRoot));
      ExpandSelectTreeNode expandSelectTreeNode = UriParser.createExpandSelectTree(uriInfo.getSelect(), uriInfo.getExpand());
      propertiesBuilder.expandSelectTree(expandSelectTreeNode).callbacks(callbacks);
      // end callback handling

      return EntityProvider.writeEntry(contentType, entitySet, data, propertiesBuilder.build());
    }
  }
...
```

For the Car it is only necessary to add the single code line below to register the additional callback and enable the `$expand`:

```java
if (ENTITY_SET_NAME_CARS.equals(entitySet.getName())) {
...
    callbacks.put(ENTITY_NAME_DRIVER, new MyCallback(dataStore, serviceRoot));
...
}
```

Deploy, run and test
Like in the basic read scenario follow these steps:

  - Build your project mvn clean install 
  - Deploy in Web Application server. When using within Eclipse simply run the Web Application via Run As -> Run on Server 
  - After successful server start and deployment the following uris from the read scenario work as before: 
    - Show the Manufacturers: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers) 
    - Show one Manufacturer: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)) 
    - Show the Cars: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars) 
    - Show one Car: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)) 
    - Show the related Manufacturer of a Car: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)/Manufacturer](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)/Manufacturer) 
    - Show the related Cars of a Manufacturer: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)/Cars ](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)/Cars )
    - Show Car with its Manufacturer: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)?$expand=Manufacturer ](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)?$expand=Manufacturer )
    - Show Manufacturer with its Cars: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)?$expand=Cars ](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)?$expand=Cars )
  - And in addition we now can access the drivers with their media resource ($value) and the association to their car. 
    - Show one Driver: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Drivers(3)](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Drivers(3)) 
    - Show Media Resource of Driver: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Drivers(3)/$value](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Drivers(3)/$value) 
    - Show Car of the Driver: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Drivers(3)/Car](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Drivers(3)/Car) 
   - Show Driver with expanded Car: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Drivers(3)?$expand=Car ](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Drivers(3)?$expand=Car )

### Conclusion
After finishing all steps of this tutorial your project contains three different entities with relations between them and one of them with media link entry and media resource support.

If something does not compile or run it is recommended to compare to the complete sample project source code in the [Olingo Tutorial 'Basic Read with Media Resource extension' Project](apache-olingo-tutorial-adv_read_mediaresource). 
For more details about how to use/setup the project in the zip see section ***Shortcut***.