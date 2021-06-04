Title: Read Scenario

# Read Scenario

---

### How To Guide for building a Sample OData service with the OData Library (Java)

This How To Guide prerequisites a Project Setup (Git, Maven, etc.) and then shows how to develop an OData Service and make the same available.
It shows in addition how to implement the Model Provider to expose the Entity Data Model (EDM) and the Data Provider to expose the runtime data.
The implementation of the Data Provider (ODataSingleProcessor) illustrates how to expose a single entry, a feed and how to follow associations.

### Prerequisites 

[Project Setup](../project-setup.html) is successfully done and the project is ready to start in your `$ODATA_PROJECT_HOME`.

### Implement your OData Service 

##### Shortcut 

As a shortcut you can download the [Olingo Tutorial 'Basic-Read' Project](apache-olingo-tutorial-basic_read.zip).

### Deployment Descriptor 

##### Sample Code      
 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="WebApp_ID" version="2.5">
    <display-name>org.apache.olingo.odata2.sample</display-name>
    <servlet>
        <servlet-name>MyODataSampleServlet</servlet-name>
        <servlet-class>org.apache.cxf.jaxrs.servlet.CXFNonSpringJaxrsServlet</servlet-class>
        <init-param>
            <param-name>javax.ws.rs.Application</param-name>
            <param-value>org.apache.olingo.odata2.core.rest.app.ODataApplication</param-value>
        </init-param>
        <init-param>
            <param-name>org.apache.olingo.odata2.service.factory</param-name>
            <param-value>org.apache.olingo.odata2.sample.service.MyServiceFactory</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>MyODataSampleServlet</servlet-name>
        <url-pattern>/MyODataSample.svc/*</url-pattern>
    </servlet-mapping>
</web-app>
```

  - Start the command line tool, go to folder *$ODATA_PROJECT_HOME\org.apache.olingo.odata2.sample.cars* and enter
`mvn clean install` to build your projects 

  - The deployment Descriptor contains two `<init-param>` elements which define the OData Application `org.apache.olingo.odata2.core.rest.app.ODataApplication` and your Service Factory `org.apache.olingo.odata2.sample.service.MyServiceFactory`. The OData Application is implemented in the OData Library (Core) and registers a root locator and an exception mapper. The root locator looks up your registered Service Factory to get access to the Entity Data Model Provider and the OData Processor which provides the runtime data. In addition the root locator looks up a parameter `org.apache.olingo.odata2.path.split` (not present in the deployment descriptor above) which indicates how many path segments are reserved for the OData Service via an Integer value (default is 0, which means that the OData Service name corresponds to the defined `url-pattern`). 

### Implement the OData Service Factory
  - Create a new source folder *src/main/java* in the eclipse project 
  - Create a new package `org.apache.olingo.odata2.sample.service` in the source folder
  - Create a class `MyServiceFactory` which extends `org.apache.olingo.odata2.api.ODataServiceFactory` in the new package and contains the following implementation 

##### Sample Code



```java
package org.apache.olingo.odata2.sample.service;
    
import org.apache.olingo.odata2.api.ODataService;
import org.apache.olingo.odata2.api.ODataServiceFactory;
import org.apache.olingo.odata2.api.edm.provider.EdmProvider;
import org.apache.olingo.odata2.api.exception.ODataException;
import org.apache.olingo.odata2.api.processor.ODataContext;
import org.apache.olingo.odata2.api.processor.ODataSingleProcessor;
    
public class MyServiceFactory extends ODataServiceFactory {

  @Override
  public ODataService createService(ODataContext ctx) throws ODataException {

    EdmProvider edmProvider = new MyEdmProvider();
    ODataSingleProcessor singleProcessor = new MyODataSingleProcessor();
    
    return createODataSingleProcessorService(edmProvider, singleProcessor);
  }
}
```

- In order to make your coding able to compile you have to create Java classes for 
 `MyEdmProvider` which extends `org.apache.olingo.odata2.api.edm.provider.EdmProvider` and 
 `MyODataSingleProcessor` which extends `org.apache.olingo.odata2.api.processor.ODataSingleProcessor` 
- After these steps compile your project with `mvn clean install` 

### Implement the Entity Data Model Provider
In this paragraph you will implement the `MyEdmProvider` class by overriding all methods of `org.apache.olingo.odata2.api.edm.provider.EdmProvider`.

- You will implement the following Entity Data Model.

![alt text][1]
 
- As we have a static model we define constants for all top level elements of the schema (declared in the `MyEdmProvider` class).

##### Sample Code

```java 
  static final String ENTITY_SET_NAME_MANUFACTURERS = "Manufacturers";
  static final String ENTITY_SET_NAME_CARS = "Cars";
  static final String ENTITY_NAME_MANUFACTURER = "Manufacturer";
  static final String ENTITY_NAME_CAR = "Car";
    
  private static final String NAMESPACE = "org.apache.olingo.odata2.ODataCars";

  private static final FullQualifiedName ENTITY_TYPE_1_1 = new FullQualifiedName(NAMESPACE, ENTITY_NAME_CAR);
  private static final FullQualifiedName ENTITY_TYPE_1_2 = new FullQualifiedName(NAMESPACE, ENTITY_NAME_MANUFACTURER);

  private static final FullQualifiedName COMPLEX_TYPE = new FullQualifiedName(NAMESPACE, "Address");

  private static final FullQualifiedName ASSOCIATION_CAR_MANUFACTURER = new FullQualifiedName(NAMESPACE, "Car_Manufacturer_Manufacturer_Cars");

  private static final String ROLE_1_1 = "Car_Manufacturer";
  private static final String ROLE_1_2 = "Manufacturer_Cars";

  private static final String ENTITY_CONTAINER = "ODataCarsEntityContainer";

  private static final String ASSOCIATION_SET = "Cars_Manufacturers";
```

- Implement `MyEdmProvider.getSchemas`. This method is used to retrieve the complete structural information in order to build the metadata document and the service document. The implementation makes use of other getter methods of this class for simplicity reasons. If a very performant way of building the whole structural information was required, other implementation strategies could be used. 

##### Sample Code

```java
public List<Schema> getSchemas() throws ODataException {
  List<Schema> schemas = new ArrayList<Schema>();
    
  Schema schema = new Schema();
  schema.setNamespace(NAMESPACE);
    
  List<EntityType> entityTypes = new ArrayList<EntityType>();
  entityTypes.add(getEntityType(ENTITY_TYPE_1_1));
  entityTypes.add(getEntityType(ENTITY_TYPE_1_2));
  schema.setEntityTypes(entityTypes);
    
  List<ComplexType> complexTypes = new ArrayList<ComplexType>();
  complexTypes.add(getComplexType(COMPLEX_TYPE));
  schema.setComplexTypes(complexTypes);
    
  List<Association> associations = new ArrayList<Association>();
  associations.add(getAssociation(ASSOCIATION_CAR_MANUFACTURER));
  schema.setAssociations(associations);
    
  List<EntityContainer> entityContainers = new ArrayList<EntityContainer>();
  EntityContainer entityContainer = new EntityContainer();
  entityContainer.setName(ENTITY_CONTAINER).setDefaultEntityContainer(true);
    
  List<EntitySet> entitySets = new ArrayList<EntitySet>();
  entitySets.add(getEntitySet(ENTITY_CONTAINER, ENTITY_SET_NAME_CARS));
  entitySets.add(getEntitySet(ENTITY_CONTAINER, ENTITY_SET_NAME_MANUFACTURERS));
  entityContainer.setEntitySets(entitySets);
    
  List<AssociationSet> associationSets = new ArrayList<AssociationSet>();
  associationSets.add(getAssociationSet(ENTITY_CONTAINER, ASSOCIATION_CAR_MANUFACTURER, ENTITY_SET_NAME_MANUFACTURERS, ROLE_1_2));
  entityContainer.setAssociationSets(associationSets);

  entityContainers.add(entityContainer);
  schema.setEntityContainers(entityContainers);

  schemas.add(schema);

  return schemas;
}
```

- `MyEdmProvider.getEntityType(FullQualifiedName edmFQName)` returns an Entity Type according to the full qualified name specified. The Entity Type holds all information about its structure like simple properties, complex properties, navigation properties and the definition of its key property (or properties). 

##### Sample Code

```java
@Override
public EntityType getEntityType(FullQualifiedName edmFQName) throws ODataException {
  if (NAMESPACE.equals(edmFQName.getNamespace())) {

    if (ENTITY_TYPE_1_1.getName().equals(edmFQName.getName())) {

      //Properties
      List<Property> properties = new ArrayList<Property>();
      properties.add(new SimpleProperty().setName("Id").setType(EdmSimpleTypeKind.Int32).setFacets(new Facets().setNullable(false)));
      properties.add(new SimpleProperty().setName("Model").setType(EdmSimpleTypeKind.String).setFacets(new Facets().setNullable(false).setMaxLength(100).setDefaultValue("Hugo"))
          .setCustomizableFeedMappings(new CustomizableFeedMappings().setFcTargetPath(EdmTargetPath.SYNDICATION_TITLE)));
      properties.add(new SimpleProperty().setName("ManufacturerId").setType(EdmSimpleTypeKind.Int32));
      properties.add(new SimpleProperty().setName("Price").setType(EdmSimpleTypeKind.Decimal));
      properties.add(new SimpleProperty().setName("Currency").setType(EdmSimpleTypeKind.String).setFacets(new Facets().setMaxLength(3)));
      properties.add(new SimpleProperty().setName("ModelYear").setType(EdmSimpleTypeKind.String).setFacets(new Facets().setMaxLength(4)));
      properties.add(new SimpleProperty().setName("Updated").setType(EdmSimpleTypeKind.DateTime)
          .setFacets(new Facets().setNullable(false).setConcurrencyMode(EdmConcurrencyMode.Fixed))
          .setCustomizableFeedMappings(new CustomizableFeedMappings().setFcTargetPath(EdmTargetPath.SYNDICATION_UPDATED)));
      properties.add(new SimpleProperty().setName("ImagePath").setType(EdmSimpleTypeKind.String));

      //Navigation Properties
      List<NavigationProperty> navigationProperties = new ArrayList<NavigationProperty>();
      navigationProperties.add(new NavigationProperty().setName("Manufacturer")
          .setRelationship(ASSOCIATION_CAR_MANUFACTURER).setFromRole(ROLE_1_1).setToRole(ROLE_1_2));

      //Key
      List<PropertyRef> keyProperties = new ArrayList<PropertyRef>();
      keyProperties.add(new PropertyRef().setName("Id"));
      Key key = new Key().setKeys(keyProperties);

      return new EntityType().setName(ENTITY_TYPE_1_1.getName())
          .setProperties(properties)
          .setKey(key)
          .setNavigationProperties(navigationProperties);

    } else if (ENTITY_TYPE_1_2.getName().equals(edmFQName.getName())) {

      //Properties
      List<Property> properties = new ArrayList<Property>();
      properties.add(new SimpleProperty().setName("Id").setType(EdmSimpleTypeKind.Int32).setFacets(new Facets().setNullable(false)));
      properties.add(new SimpleProperty().setName("Name").setType(EdmSimpleTypeKind.String).setFacets(new Facets().setNullable(false).setMaxLength(100))
          .setCustomizableFeedMappings(new CustomizableFeedMappings().setFcTargetPath(EdmTargetPath.SYNDICATION_TITLE)));
      properties.add(new ComplexProperty().setName("Address").setType(new FullQualifiedName(NAMESPACE, "Address")));
      properties.add(new SimpleProperty().setName("Updated").setType(EdmSimpleTypeKind.DateTime)
          .setFacets(new Facets().setNullable(false).setConcurrencyMode(EdmConcurrencyMode.Fixed))
          .setCustomizableFeedMappings(new CustomizableFeedMappings().setFcTargetPath(EdmTargetPath.SYNDICATION_UPDATED)));

      //Navigation Properties
      List<NavigationProperty> navigationProperties = new ArrayList<NavigationProperty>();
      navigationProperties.add(new NavigationProperty().setName("Cars")
          .setRelationship(ASSOCIATION_CAR_MANUFACTURER).setFromRole(ROLE_1_2).setToRole(ROLE_1_1));

      //Key
      List<PropertyRef> keyProperties = new ArrayList<PropertyRef>();
      keyProperties.add(new PropertyRef().setName("Id"));
      Key key = new Key().setKeys(keyProperties);

      return new EntityType().setName(ENTITY_TYPE_1_2.getName())
          .setProperties(properties)
          .setHasStream(true)
          .setKey(key)
          .setNavigationProperties(navigationProperties);
    }
  }

  return null;
}
```

- `MyEdmProvider.getComplexType(FullQualifiedName edmFQName)` 

##### Sample Code


```java
public ComplexType getComplexType(FullQualifiedName edmFQName) throws ODataException {
  if (NAMESPACE.equals(edmFQName.getNamespace())) {
    if (COMPLEX_TYPE.getName().equals(edmFQName.getName())) {
      List<Property> properties = new ArrayList<Property>();
      properties.add(new SimpleProperty().setName("Street").setType(EdmSimpleTypeKind.String));
      properties.add(new SimpleProperty().setName("City").setType(EdmSimpleTypeKind.String));
      properties.add(new SimpleProperty().setName("ZipCode").setType(EdmSimpleTypeKind.String));
      properties.add(new SimpleProperty().setName("Country").setType(EdmSimpleTypeKind.String));
      return new ComplexType().setName(COMPLEX_TYPE.getName()).setProperties(properties);
    }
  }

  return null;

}
```  

  


- `MyEdmProvider.getAssociation(FullQualifiedName edmFQName)` 
  
##### Sample Code


```java
public Association getAssociation(FullQualifiedName edmFQName) throws ODataException {
  if (NAMESPACE.equals(edmFQName.getNamespace())) {
    if (ASSOCIATION_CAR_MANUFACTURER.getName().equals(edmFQName.getName())) {
      return new Association().setName(ASSOCIATION_CAR_MANUFACTURER.getName())
          .setEnd1(new AssociationEnd().setType(ENTITY_TYPE_1_1).setRole(ROLE_1_1).setMultiplicity(EdmMultiplicity.MANY))
          .setEnd2(new AssociationEnd().setType(ENTITY_TYPE_1_2).setRole(ROLE_1_2).setMultiplicity(EdmMultiplicity.ONE));
    }
  }
  return null;
}
```

- `MyEdmProvider.getEntityContainerInfo(String name)` 
  
##### Sample Code

```java
public EntityContainerInfo getEntityContainerInfo(String name) throws ODataException {
  if (name == null || "ODataCarsEntityContainer".equals(name)) {
    return new EntityContainerInfo().setName("ODataCarsEntityContainer").setDefaultEntityContainer(true);
  }

  return null;
}
```

- `MyEdmProvider.getEntitySet(String entityContainer, String name)`

##### Sample Code

```java
public EntitySet getEntitySet(String entityContainer, String name) throws ODataException {
  if (ENTITY_CONTAINER.equals(entityContainer)) {
    if (ENTITY_SET_NAME_CARS.equals(name)) {
      return new EntitySet().setName(name).setEntityType(ENTITY_TYPE_1_1);
    } else if (ENTITY_SET_NAME_MANUFACTURERS.equals(name)) {
      return new EntitySet().setName(name).setEntityType(ENTITY_TYPE_1_2);
    }
  }
  return null;
}
```

- `MyEdmProvider.getAssociationSet(String entityContainer, FullQualifiedName association, String sourceEntitySetName, String sourceEntitySetRole)`

##### Sample Code

 
```java
public AssociationSet getAssociationSet(String entityContainer, FullQualifiedName association, String sourceEntitySetName, String sourceEntitySetRole) throws ODataException {
  if (ENTITY_CONTAINER.equals(entityContainer)) {
    if (ASSOCIATION_CAR_MANUFACTURER.equals(association)) {
      return new AssociationSet().setName(ASSOCIATION_SET)
          .setAssociation(ASSOCIATION_CAR_MANUFACTURER)
          .setEnd1(new AssociationSetEnd().setRole(ROLE_1_2).setEntitySet(ENTITY_SET_NAME_MANUFACTURERS))
          .setEnd2(new AssociationSetEnd().setRole(ROLE_1_1).setEntitySet(ENTITY_SET_NAME_CARS));
    }
  }
  return null;
}
```

#### Conclusion

After the implementation of the Edm Provider the web application can be executed to show the Service Document and the Metadata Document.

- Build your project `mvn clean install` 
- Deploy the Web Application to the server. 
 - Show the Service Document: http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/
 - Show the Metadata Document: http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/$metadata 

### Implement the OData Processor which provides the runtime data

You already created the `MyODataSingleProcessor` class which we now extend with some needed imports and a reference to a DataStore which contains our data (and will be implemented in the next step).

##### Sample Code
 
```java
package org.apache.olingo.odata2.sample.service;

import static org.apache.olingo.odata2.sample.service.MyEdmProvider.ENTITY_SET_NAME_CARS;
import static org.apache.olingo.odata2.sample.service.MyEdmProvider.ENTITY_SET_NAME_MANUFACTURERS;

import org.apache.olingo.odata2.api.processor.ODataSingleProcessor;

public class MyODataSingleProcessor extends ODataSingleProcessor {
  private DataStore dataStore = new DataStore();
}
```

- As next steps we will implement the read access to the Car and Manufacturer entries and the read access to the Cars and Manufacturers feed. As we need some basis for sample data we create a very simple DataStore which contains the data as well as access methods to serve the required data: 

##### Sample Code


```java
package org.apache.olingo.odata2.sample.service;
    
import java.util.ArrayList;
import java.util.Calendar;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.TimeZone;

public class DataStore {

  //Data accessors
  public Map<String, Object> getCar(int id) {
    Map<String, Object> data = null;

    Calendar updated = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
    
    switch (id) {
    case 1:
      updated.set(2012, 11, 11, 11, 11, 11);
      data = createCar(1, "F1 W03", 1, 189189.43, "EUR", "2012", updated, "file://imagePath/w03");
      break;

    case 2:
      updated.set(2013, 11, 11, 11, 11, 11);
      data = createCar(2, "F1 W04", 1, 199999.99, "EUR", "2013", updated, "file://imagePath/w04");
      break;

    case 3:
      updated.set(2012, 12, 12, 12, 12, 12);
      data = createCar(3, "F2012", 2, 137285.33, "EUR", "2012", updated, "http://pathToImage/f2012");
      break;

    case 4:
      updated.set(2013, 12, 12, 12, 12, 12);
      data = createCar(4, "F2013", 2, 145285.00, "EUR", "2013", updated, "http://pathToImage/f2013");
      break;

    case 5:
      updated.set(2011, 11, 11, 11, 11, 11);
      data = createCar(5, "F1 W02", 1, 167189.00, "EUR", "2011", updated, "file://imagePath/wXX");
      break;

    default:
      break;
    }
    
    return data;
  }
      
  private Map<String, Object> createCar(int carId, String model, int manufacturerId, double price, String currency, String modelYear, Calendar updated, String imagePath) {
    Map<String, Object> data = new HashMap<String, Object>();
    
    data.put("Id", carId);
    data.put("Model", model);
    data.put("ManufacturerId", manufacturerId);
    data.put("Price", price);
    data.put("Currency", currency);
    data.put("ModelYear", modelYear);
    data.put("Updated", updated);
    data.put("ImagePath", imagePath);
    
    return data;
  }
      
  public Map<String, Object> getManufacturer(int id) {
    Map<String, Object> data = null;
    Calendar date = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
    
    switch (id) {
    case 1:
      Map<String, Object> addressStar = createAddress("Star Street 137", "Stuttgart", "70173", "Germany");
      date.set(1954, 7, 4);
      data = createManufacturer(1, "Star Powered Racing", addressStar, date);
      break;
      
    case 2:
      Map<String, Object> addressHorse = createAddress("Horse Street 1", "Maranello", "41053", "Italy");
      date.set(1929, 11, 16);
      data = createManufacturer(2, "Horse Powered Racing", addressHorse, date);
      break;
      
    default:
      break;
    }
    
    return data;
  }
    
  private Map<String, Object> createManufacturer(int id, String name, Map<String, Object> address, Calendar updated) {
    Map<String, Object> data = new HashMap<String, Object>();
    data.put("Id", id);
    data.put("Name", name);
    data.put("Address", address);
    data.put("Updated", updated);
    return data;
  }
      
  private Map<String, Object> createAddress(String street, String city, String zipCode, String country) {
    Map<String, Object> address = new HashMap<String, Object>();
    address.put("Street", street);
    address.put("City", city);
    address.put("ZipCode", zipCode);
    address.put("Country", country);
    return address;
  }
    
    
  public List<Map<String, Object>> getCars() {
    List<Map<String, Object>> cars = new ArrayList<Map<String, Object>>();
    cars.add(getCar(1));
    cars.add(getCar(2));
    cars.add(getCar(3));
    cars.add(getCar(4));
    cars.add(getCar(5));
    return cars;
  }
      
  public List<Map<String, Object>> getManufacturers() {
    List<Map<String, Object>> manufacturers = new ArrayList<Map<String, Object>>();
    manufacturers.add(getManufacturer(1));
    manufacturers.add(getManufacturer(2));
    return manufacturers;
  }
    

  public List<Map<String, Object>> getCarsFor(int manufacturerId) {
    List<Map<String, Object>> cars = getCars();
    List<Map<String, Object>> carsForManufacturer = new ArrayList<Map<String,Object>>();
    
    for (Map<String,Object> car: cars) {
      if(Integer.valueOf(manufacturerId).equals(car.get("ManufacturerId"))) {
        carsForManufacturer.add(car);
      }
    }
    
    return carsForManufacturer;
  }
   
  public Map<String, Object> getManufacturerFor(int carId) {
    Map<String, Object> car = getCar(carId);
    if(car != null) {
      Object manufacturerId = car.get("ManufacturerId");
      if(manufacturerId != null) {
        return getManufacturer((Integer) manufacturerId);
      }
    }
    return null;
  }
}
```





- Implement `MyODataSingleProcessor.readEntity(GetEntityUriInfo uriParserResultInfo)` by overriding the corresponding method of the ODataSingleProcessor

##### Sample Code
 
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
          
          return EntityProvider.writeEntry(contentType, entitySet, data, propertiesBuilder.build());
        }
      } else if (ENTITY_SET_NAME_MANUFACTURERS.equals(entitySet.getName())) {
        int id = getKeyValue(uriInfo.getKeyPredicates().get(0));
        Map<String, Object> data = dataStore.getManufacturer(id);
        
        if (data != null) {
          URI serviceRoot = getContext().getPathInfo().getServiceRoot();
          ODataEntityProviderPropertiesBuilder propertiesBuilder = EntityProviderWriteProperties.serviceRoot(serviceRoot);
  
          return EntityProvider.writeEntry(contentType, entitySet, data, propertiesBuilder.build());
        }
      }

      throw new ODataNotFoundException(ODataNotFoundException.ENTITY);

    } else if (uriInfo.getNavigationSegments().size() == 1) {
      //navigation first level, simplified example for illustration purposes only
      EdmEntitySet entitySet = uriInfo.getTargetEntitySet();
      if (ENTITY_SET_NAME_MANUFACTURERS.equals(entitySet.getName())) {
        int carKey = getKeyValue(uriInfo.getKeyPredicates().get(0));
        return EntityProvider.writeEntry(contentType, uriInfo.getTargetEntitySet(), dataStore.getManufacturer(carKey), EntityProviderWriteProperties.serviceRoot(getContext().getPathInfo().getServiceRoot()).build());
      }

      throw new ODataNotFoundException(ODataNotFoundException.ENTITY);
    }

    throw new ODataNotImplementedException();
  }
  Implement MyODataSingleProcessor.readEntitySet(GetEntitySetUriInfo uriParserResultInfo) by overriding the corresponding method of the ODataSingleProcessor 
    public ODataResponse readEntitySet(GetEntitySetUriInfo uriInfo, String contentType) throws ODataException {
    
    EdmEntitySet entitySet;

    if (uriInfo.getNavigationSegments().size() == 0) {
      entitySet = uriInfo.getStartEntitySet();

      if (ENTITY_SET_NAME_CARS.equals(entitySet.getName())) {
        return EntityProvider.writeFeed(contentType, entitySet, dataStore.getCars(), EntityProviderWriteProperties.serviceRoot(getContext().getPathInfo().getServiceRoot()).build());
      } else if (ENTITY_SET_NAME_MANUFACTURERS.equals(entitySet.getName())) {
        return EntityProvider.writeFeed(contentType, entitySet, dataStore.getManufacturers(), EntityProviderWriteProperties.serviceRoot(getContext().getPathInfo().getServiceRoot()).build());
      }

      throw new ODataNotFoundException(ODataNotFoundException.ENTITY);

    } else if (uriInfo.getNavigationSegments().size() == 1) {
      //navigation first level, simplified example for illustration purposes only
      entitySet = uriInfo.getTargetEntitySet();

      if (ENTITY_SET_NAME_CARS.equals(entitySet.getName())) {
        int manufacturerKey = getKeyValue(uriInfo.getKeyPredicates().get(0));

        List<Map<String, Object>> cars = new ArrayList<Map<String, Object>>();
        cars.add(dataStore.getCar(manufacturerKey));

        return EntityProvider.writeFeed(contentType, entitySet, cars, EntityProviderWriteProperties.serviceRoot(getContext().getPathInfo().getServiceRoot()).build());
      }

      throw new ODataNotFoundException(ODataNotFoundException.ENTITY);
    }

    throw new ODataNotImplementedException();
  }
```

And add the small method to get the key value of a `KeyPredicate`:

```java
  private int getKeyValue(KeyPredicate key) throws ODataException {
    EdmProperty property = key.getProperty();
    EdmSimpleType type = (EdmSimpleType) property.getType();
    return type.valueOfString(key.getLiteral(), EdmLiteralKind.DEFAULT, property.getFacets(), Integer.class);
  }
```

After the implementation of the `MyODataSingleProcessor` the web application can be tested.

- Build your project. Remember? `mvn clean install`
- Deploy the web application on your local server
- Show the Manufacturers: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers)
- Show one Manufacturer: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1))
- Show the Cars: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars)
- Show one Car: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2))
- Show the related Manufacturer of a Car: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)/Manufacturer](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Cars(2)/Manufacturer)
- Show the related Cars of a Manufacturer: [http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)/Cars](http://localhost:8080/olingo.odata2.sample.cars.web/MyODataSample.svc/Manufacturers(1)/Cars)


  [1]: /img/ODataCarsModel.JPG