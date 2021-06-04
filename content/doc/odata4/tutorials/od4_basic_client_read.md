Title: Client Scenario

# Client Scenario

---

## How To Guide for building a Sample OData Client with the OData 4.0 Library (for Java)

This Tutorial shows how to use the Apache Olingo Library for CRUD operations on an existing OData Service.
Therefore it contains the Explaining the Client section which explains how to implement the CRUD operations based on sample code.
For creating a simple odata service refer the section Basic Tutorial: Create an OData V4 Service with Olingo in Olingo V4 tutorial
###Client Quickstart Guide
With this Quickstart guide the runnable sample client and an sample service is created within a few minutes. Therefore it just requires an installed Java 6 Runtime, Maven 3 and an internet connection.
It also requires an odata service. This sample uses the Cars service which can be found under samples/server under olingo [git repository][1]. Build the project and deploy the war on a server. Follow the [Guide - To fetch the tutorial sources][2] to import the sample server project.

 1. Create a sample maven project and name it OlingoSampleApp.
 2. Create a new package org.apache.olingo.samples.client under this
    project.
 3. Create a class OlingoSampleApp.java under package
    org.apache.olingo.samples.client.
 4. Copy the entire [sample client code](#sample) into this class.
 5. Copy [this sample pom into](#pom) into this projects pom.xml file.
 6. Run OlingoSampleApp against sample Service
 7. In order to fetch all the dependencies in the pom.xml file run
    eclipse:eclipse command on the sample client project

### Explaining the Client
### Create OData Client 

    public static final ODataClient client = ODataClientFactory.getClient();
    odataClient.getConfiguration().setDefaultPubFormat(ContentType.APPLICATION_JSON);

### Read EDM
For an OData Service the Entity Data Model (EDM) defines all metadata information about the provided data of the service. This includes all entities with their type, properties and relations, which entities are provided as entity sets and additional functions and operations provided by the OData Service. The EDM also have to be provided by the OData Service via a unique URI (e.g. http://localhost:8080/cars.svc/$metadata) in the EDMX format.
This fact is important because the Apache Olingo library requires the metadata for serialization and de-serialization of the data of an entity (e.g. the validation of the data is done against the EDM provided metadata). Hence the first step in this sample is to read the whole EDM of an OData Service.
###Code sample: Read EDM ($metadata)

    final Edm edm = getClient().getRetrieveRequestFactory().getMetadataRequest(serviceUrl).execute().getBody();
    return edm;

If annotations defined in external vocabulary file has to be loaded then the below code has to be used

    List<InputStream> streams = new ArrayList<InputStream>();
    //If file is locally available    
    streams.add(getClass().getResourceAsStream("annotations.xml"));
    XMLMetadata xmlMetadata = getClient().getRetrieveRequestFactory().getXMLMetadataRequest(serviceUrl).execute().getBody();
    //If the reference uri's have to be loaded 
    String vocabUrl = metadata.getReferences().get(0).getUri().toString();
    URI uri = new URI(vocabUrl);
    ODataRawRequest request = getClient().getRetrieveRequestFactory().getRawRequest(uri);
    ODataRawResponse response = request.execute();
    streams.add(response.getRawResponse());
    final Edm edm = getClient().getReader().readMetadata(xmlMetadata, streams);
    return edm;

Here the serviceUrl is the root Url of the odata service.
For read and de-serialize of the EDM this is all what have to be done and the resulting EDM instance than can be used for necessary serialization and de-serialization in combination of CRUD operations supported by Apache Olingo library.

### Read Entity
For reading entities this sample provides two methods. First is read of a complete collection / Entity Set and second is read of a single Entity. In general, for both first create the request and execute the request.

### Read entityCollection

    ODataEntitySetRequest<ClientEntitySet> request = getClient().getRetrieveRequestFactory()
            .getEntitySetRequest(getClient().newURIBuilder(serviceUrl)
            .appendEntitySetSegment(“Manufacturers”).build());
    final ODataRetrieveResponse<ClientEntitySet> response = request.execute();
    final ClientEntitySet entitySet = response.getBody();

For read of a complete collection the request URI is a EntitySet. Via the execute method the request is done against the created uri and the responding content is returned as HttpResponse. This HttpResponse then will be de-serialized by the library into an ClientEntitySet object which contains all entities, each entities navigation links provided by the OData Service.

### Read Entity

    ODataEntityRequest<ClientEntity> request = getClient().getRetrieveRequestFactory()
            .getEntityRequest(getClient().newURIBuilder(serviceUrl)
            .appendEntitySetSegment(“Manufacturers”).appendKeySegment(1).build());
    final ODataRetrieveResponse<ClientEntity> response = request.execute();
    final ClientEntity entity = response.getBody();

For read of a single ODataEntry the request URI is an Entity for which a key value is required for creation of the uri. Via the execute method the request is done against the created uri and the responding content is returned as HttpResponse. This HttpResponse then will be de-serialized by the library into an ClientEntity object which contains all properties of an entity, along with navigation links provided by the OData Service.

### Read Entity Property

    ODataPropertyRequest<ClientProperty> request = getClient().getRetrieveRequestFactory()
            .getPropertyRequest(odataClient.newURIBuilder(serviceUrl)
            .appendEntitySetSegment(“Manufacturers”).appendKeySegment(1)
            .appendPropertySegment(“Name”).build());
    final ODataRetrieveResponse<ClientProperty> response = request.execute();
    final ClientProperty property = response.getBody();
    //If property is a primitive type and if value has to be fetched
    final ClientProperty property = property.get("Name");
    final ClientPrimitiveValue clientValue = property.getPrimitiveValue();
    //If the property is a Complex Type and if value has to be fetched
    // Here Address is a complex property
    final ClientComplexValue complexValue = prop.getComplexValue();
    final ClientValue propertyComp = complexValue.get("Street").getValue();

### Create Entity

To create an entity a HTTP POST on the corresponding entity set URI with the whole entity data as POST Body in a supported format (e.g. atom-xml, json) has to be done. With Apache Olingo the required POST Body can be created (serialized) with the methods available on ClientObjectFactory. This method creates a ClientEntity which contains the content (i.e. the required POST Body) which then can be send to the server. If the entry was created successfully an HTTP Status: 201 created will be returned as well as the complete entry.
For simplicity in the code sample below the prepare and execute of the POST and the read of the response is separated (see Part 1: Post and Part 2: Read).
### Code sample: Create single Entry
### Part 1: POST entry

    ClientEntity newEntity = getClient().getObjectFactory().newEntity(“Manufacturers”);
    newEntity.getProperties().add(getClient().getObjectFactory().newPrimitiveProperty(“Name”,
            getFactory().newPrimitiveValueBuilder().buildString(“MyCarManufacturer”)));
    newEntity.addLink(getClient().getObjectFactory().newEntityNavigationLink(“Cars”,
            client.newURIBuilder(serviceUrl)
                .appendEntitySetSegment(“Cars”)
                .appendKeySegment(1)
                .build()));

With the ODataClientFactory it is possible to create a new entity along with its properties, its values and also links.

### Part 2: Read response

    final ODataEntityCreateRequest<ClientEntity> createRequest = getClient().getCUDRequestFactory().getEntityCreateRequest(
            getClient().newURIBuilder(serviceUrl).appendEntitySetSegment(“Manufacturers”).build(),
            newEntity);
    final ODataEntityCreateResponse<ClientEntity> createResponse = createRequest.execute();
    final ClientEntity createdEntity = createResponse.getBody();

This executes the create request and the response will return the ClientEntity that was created.
###PUT entry

    final URI uri = odataClient.newURIBuilder(serviceUrl)
            .appendEntitySetSegment(“Manufacturers”).appendKeySegment(1).build();
    
    final ClientEntity entity = getClient().getObjectFactory().newEntity(new FullQualifiedName(“OData.Demo.Manufacturer”));
    entity.getProperties().add(getClient().getObjectFactory().newPrimitiveProperty(“Name”,
            getClient().getObjectFactory().newPrimitiveValueBuilder().buildString(“MyCarManufacturer”)));
    
    final ODataEntityUpdateRequest<ClientEntity> requestUpdate = odataClient.getCUDRequestFactory().getEntityUpdateRequest(uri,  UpdateType.PATCH, entity);    
    final ODataEntityUpdateResponse<ClientEntity> responseUpdate = requestUpdate.execute();

With the ODataClientFactory create the entity that has to be updated along with its properties and its values. Then execute the update request and the response will be the ClientEntity with no content.
If the entry was updated successfully an HTTP Status: 204 No content will be returned.

### Code sample: Delete single Entry
### Delete
For deletion of an entry just a DELETE request is necessary on the URI of the entity. Hence the Apache Olingo is not necessary to serialize or de-serialize anything for this use case.

    final URI uri = getClient().newURIBuilder(serviceUrl).appendEntitySetSegment(“Manufacturers”).appendKeySegment(1).build();
    final ODataDeleteRequest request = getClient().getCUDRequestFactory().getDeleteRequest(uri);
    final ODataDeleteResponse response = request.execute();

So the code for delete of an entry the  DELETE request URI is an Entity for which a key value is required for creation of the absolut uri. Via execute() method the request is done against the uri and the responding http status code is returned, which is, if the entry was deleted successfully, an HTTP Status: 204 No content.

<a name="pom"></a>

### Sample pom.xml file

    <?xml version="1.0" encoding="UTF-8"?>
    <!--
    
        Licensed to the Apache Software Foundation (ASF) under one
        or more contributor license agreements.  See the NOTICE file
        distributed with this work for additional information
        regarding copyright ownership.  The ASF licenses this file
        to you under the Apache License, Version 2.0 (the
        "License"); you may not use this file except in compliance
        with the License.  You may obtain a copy of the License at
    
          http://www.apache.org/licenses/LICENSE-2.0
    
        Unless required by applicable law or agreed to in writing,
        software distributed under the License is distributed on an
        "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
        KIND, either express or implied.  See the License for the
        specific language governing permissions and limitations
        under the License.
    
    -->
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    
      <modelVersion>4.0.0</modelVersion>
    
      <artifactId>odata-client-sample</artifactId>
      <packaging>jar</packaging>
      <name>${project.artifactId}</name>
    
      <parent>
        <groupId>org.apache.olingo</groupId>
        <artifactId>odata-samples</artifactId>
        <version>4.5.0-sap-05-SNAPSHOT</version>
        <relativePath>..</relativePath>
      </parent>
    
      <build>
        <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-deploy-plugin</artifactId>
            <configuration>
              <skip>true</skip>
            </configuration>
          </plugin>
    
          <!-- Disable checkstyle for sample -->
          <plugin>
            <artifactId>maven-checkstyle-plugin</artifactId>
            <configuration>
              <skip>true</skip>
            </configuration>
          </plugin>
        </plugins>
      </build>
    
      <dependencies>
        <dependency>
          <groupId>org.apache.olingo</groupId>
          <artifactId>odata-client-core</artifactId>
          <version>${project.version}</version>
        </dependency>
        <dependency>
          <groupId>commons-logging</groupId>
          <artifactId>commons-logging</artifactId>
          <version>${commons.logging.version}</version>
          <scope>runtime</scope>
        </dependency>
        <dependency>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-simple</artifactId>
          <version>${sl4j.version}</version>
          <scope>runtime</scope>
        </dependency>
      </dependencies>
      <profiles>
        <profile>
          <id>client</id>
          <build>
            <defaultGoal>test</defaultGoal>
            <plugins>
              <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <executions>
                  <execution>
                    <phase>test</phase>
                    <goals>
                      <goal>java</goal>
                    </goals>
                    <configuration>
                      <mainClass>org.apache.olingo.samples.client.OlingoSampleApp</mainClass>
                    </configuration>
                  </execution>
                </executions>
              </plugin>
            </plugins>
          </build>
        </profile>
      </profiles>
    </project>

<a name="sample"></a>

### OlingoSampleApp.java

    /*
     * Licensed to the Apache Software Foundation (ASF) under one
     * or more contributor license agreements. See the NOTICE file
     * distributed with this work for additional information
     * regarding copyright ownership. The ASF licenses this file
     * to you under the Apache License, Version 2.0 (the
     * "License"); you may not use this file except in compliance
     * with the License. You may obtain a copy of the License at
     * 
     * http://www.apache.org/licenses/LICENSE-2.0
     * 
     * Unless required by applicable law or agreed to in writing,
     * software distributed under the License is distributed on an
     * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
     * KIND, either express or implied. See the License for the
     * specific language governing permissions and limitations
     * under the License.
     ******************************************************************************/
    package org.apache.olingo.samples.client;
    
    import java.io.IOException;
    import java.io.InputStream;
    import java.net.URI;
    import java.text.SimpleDateFormat;
    import java.util.ArrayList;
    import java.util.Calendar;
    import java.util.Collection;
    import java.util.List;
    import java.util.Map;
    import java.util.Map.Entry;
    import java.util.Set;
    
    import org.apache.olingo.client.api.ODataClient;
    import org.apache.olingo.client.api.communication.request.cud.ODataDeleteRequest;
    import org.apache.olingo.client.api.communication.request.cud.ODataEntityCreateRequest;
    import org.apache.olingo.client.api.communication.request.cud.ODataEntityUpdateRequest;
    import org.apache.olingo.client.api.communication.request.cud.UpdateType;
    import org.apache.olingo.client.api.communication.request.retrieve.EdmMetadataRequest;
    import org.apache.olingo.client.api.communication.request.retrieve.ODataEntityRequest;
    import org.apache.olingo.client.api.communication.request.retrieve.ODataEntitySetIteratorRequest;
    import org.apache.olingo.client.api.communication.response.ODataDeleteResponse;
    import org.apache.olingo.client.api.communication.response.ODataEntityCreateResponse;
    import org.apache.olingo.client.api.communication.response.ODataEntityUpdateResponse;
    import org.apache.olingo.client.api.communication.response.ODataRetrieveResponse;
    import org.apache.olingo.client.api.domain.ClientCollectionValue;
    import org.apache.olingo.client.api.domain.ClientComplexValue;
    import org.apache.olingo.client.api.domain.ClientEntity;
    import org.apache.olingo.client.api.domain.ClientEntitySet;
    import org.apache.olingo.client.api.domain.ClientEntitySetIterator;
    import org.apache.olingo.client.api.domain.ClientEnumValue;
    import org.apache.olingo.client.api.domain.ClientProperty;
    import org.apache.olingo.client.api.domain.ClientValue;
    import org.apache.olingo.client.api.serialization.ODataDeserializerException;
    import org.apache.olingo.client.core.ODataClientFactory;
    import org.apache.olingo.commons.api.edm.Edm;
    import org.apache.olingo.commons.api.edm.EdmComplexType;
    import org.apache.olingo.commons.api.edm.EdmEntityType;
    import org.apache.olingo.commons.api.edm.EdmProperty;
    import org.apache.olingo.commons.api.edm.EdmSchema;
    import org.apache.olingo.commons.api.edm.FullQualifiedName;
    import org.apache.olingo.commons.api.format.ContentType;
    
    
    /**
     *
     */
    public class OlingoSampleApp {
      private ODataClient client;
      
      public OlingoSampleApp() {
        client = ODataClientFactory.getClient();
      }
    
      public static void main(String[] params) throws Exception {
        OlingoSampleApp app = new OlingoSampleApp();
        app.perform("http://localhost:8080/cars.svc");
      }
    
      void perform(String serviceUrl) throws Exception {
    
        print("\n----- Read Edm ------------------------------");
        Edm edm = readEdm(serviceUrl);
        List<FullQualifiedName> ctFqns = new ArrayList<FullQualifiedName>();
        List<FullQualifiedName> etFqns = new ArrayList<FullQualifiedName>();
        for (EdmSchema schema : edm.getSchemas()) {
          for (EdmComplexType complexType : schema.getComplexTypes()) {
            ctFqns.add(complexType.getFullQualifiedName());
          }
          for (EdmEntityType entityType : schema.getEntityTypes()) {
            etFqns.add(entityType.getFullQualifiedName());
          }
        }
        print("Found ComplexTypes", ctFqns);
        print("Found EntityTypes", etFqns);
    
        print("\n----- Inspect each property and its type of the first entity: " + etFqns.get(0) + "----");
        EdmEntityType etype = edm.getEntityType(etFqns.get(0));
        for (String propertyName : etype.getPropertyNames()) {
          EdmProperty property = etype.getStructuralProperty(propertyName);
          FullQualifiedName typeName = property.getType().getFullQualifiedName();
          print("property '" + propertyName + "' " + typeName);
        }
        
        print("\n----- Read Entities ------------------------------");
        ClientEntitySetIterator<ClientEntitySet, ClientEntity> iterator = 
          readEntities(edm, serviceUrl, "Manufacturers");
    
        while (iterator.hasNext()) {
          ClientEntity ce = iterator.next();
          print("Entry:\n" + prettyPrint(ce.getProperties(), 0));
        }
    
        print("\n----- Read Entry ------------------------------");
        ClientEntity entry = readEntityWithKey(edm, serviceUrl, "Manufacturers", 1);
        print("Single Entry:\n" + prettyPrint(entry.getProperties(), 0));
    
        //
        print("\n----- Read Entity with $expand  ------------------------------");
        entry = readEntityWithKeyExpand(edm, serviceUrl, "Manufacturers", 1, "Cars");
        print("Single Entry with expanded Cars relation:\n" + prettyPrint(entry.getProperties(), 0));
    
        //
        print("\n----- Read Entities with $filter  ------------------------------");
        iterator = readEntitiesWithFilter(edm, serviceUrl, "Manufacturers", "Name eq 'Horse Powered Racing'");
        while (iterator.hasNext()) {
          ClientEntity ce = iterator.next();
          print("Entry:\n" + prettyPrint(ce.getProperties(), 0));
        }
    
        // skip everything as odata4 sample/server only supporting retrieval
        print("\n----- Create Entry ------------------------------");
        ClientEntity ce = loadEntity("/mymanufacturer.json");
        entry = createEntity(edm, serviceUrl, "Manufacturers", ce);
    
        print("\n----- Update Entry ------------------------------");
        ce = loadEntity("/mymanufacturer2.json");
        int sc = updateEntity(edm, serviceUrl, "Manufacturers", 123, ce);
        print("Updated successfully: " + sc);
        entry = readEntityWithKey(edm, serviceUrl, "Manufacturers", 123);
        print("Updated Entry successfully: " + prettyPrint(entry.getProperties(), 0));
    
    
        print("\n----- Delete Entry ------------------------------");
        sc = deleteEntity(serviceUrl, "Manufacturers", 123);
        print("Deletion of Entry was successfully: " + sc);
    
        try {
          print("\n----- Verify Delete Entry ------------------------------");
          readEntityWithKey(edm, serviceUrl, "Manufacturers", 123);
        } catch(Exception e) {
          print(e.getMessage());
        }
      }
    
      private static void print(String content) {
        System.out.println(content);
      }
    
      private static void print(String content, List<?> list) {
        System.out.println(content);
        for (Object o : list) {
            System.out.println("    " + o);
        }
          System.out.println();
      }
    
      private static String prettyPrint(Map<String, Object> properties, int level) {
        StringBuilder b = new StringBuilder();
        Set<Entry<String, Object>> entries = properties.entrySet();
    
        for (Entry<String, Object> entry : entries) {
          intend(b, level);
          b.append(entry.getKey()).append(": ");
          Object value = entry.getValue();
          if(value instanceof Map) {
            value = prettyPrint((Map<String, Object>) value, level+1);
          } else if(value instanceof Calendar) {
            Calendar cal = (Calendar) value;
            value = SimpleDateFormat.getInstance().format(cal.getTime());
          }
          b.append(value).append("\n");
        }
        // remove last line break
        b.deleteCharAt(b.length()-1);
        return b.toString();
      }
      
      private static String prettyPrint(Collection<ClientProperty> properties, int level) {
        StringBuilder b = new StringBuilder();
    
        for (ClientProperty entry : properties) {
          intend(b, level);
          ClientValue value = entry.getValue();
          if (value.isCollection()) {
            ClientCollectionValue cclvalue = value.asCollection();
            b.append(prettyPrint(cclvalue.asJavaCollection(), level + 1));
          } else if (value.isComplex()) {
            ClientComplexValue cpxvalue = value.asComplex();
            b.append(prettyPrint(cpxvalue.asJavaMap(), level + 1));
          } else if (value.isEnum()) {
            ClientEnumValue cnmvalue = value.asEnum();
            b.append(entry.getName()).append(": ");
            b.append(cnmvalue.getValue()).append("\n");
          } else if (value.isPrimitive()) {
            b.append(entry.getName()).append(": ");
            b.append(entry.getValue()).append("\n");
          }
        }
        return b.toString();
      }
    
      private static void intend(StringBuilder builder, int intendLevel) {
        for (int i = 0; i < intendLevel; i++) {
          builder.append("  ");
        }
      }
    
      public Edm readEdm(String serviceUrl) throws IOException {
        EdmMetadataRequest request = client.getRetrieveRequestFactory().getMetadataRequest(serviceUrl);
        ODataRetrieveResponse<Edm> response = request.execute();
        return response.getBody();
      }
    
      public ClientEntitySetIterator<ClientEntitySet, ClientEntity> readEntities(Edm edm, String serviceUri,
        String entitySetName) {
        URI absoluteUri = client.newURIBuilder(serviceUri).appendEntitySetSegment(entitySetName).build();
        return readEntities(edm, absoluteUri);
      }
    
      public ClientEntitySetIterator<ClientEntitySet, ClientEntity> readEntitiesWithFilter(Edm edm, String serviceUri,
        String entitySetName, String filterName) {
        URI absoluteUri = client.newURIBuilder(serviceUri).appendEntitySetSegment(entitySetName).filter(filterName).build();
        return readEntities(edm, absoluteUri);
      }
    
      private ClientEntitySetIterator<ClientEntitySet, ClientEntity> readEntities(Edm edm, URI absoluteUri) {
        System.out.println("URI = " + absoluteUri);
        ODataEntitySetIteratorRequest<ClientEntitySet, ClientEntity> request = 
          client.getRetrieveRequestFactory().getEntitySetIteratorRequest(absoluteUri);
        request.setAccept("application/json");
        ODataRetrieveResponse<ClientEntitySetIterator<ClientEntitySet, ClientEntity>> response = request.execute(); 
          
        return response.getBody();
      }
    
      public ClientEntity readEntityWithKey(Edm edm, String serviceUri, String entitySetName, Object keyValue) {
        URI absoluteUri = client.newURIBuilder(serviceUri).appendEntitySetSegment(entitySetName)
          .appendKeySegment(keyValue).build();
        return readEntity(edm, absoluteUri);
      }
    
      public ClientEntity readEntityWithKeyExpand(Edm edm, String serviceUri, String entitySetName, Object keyValue,
        String expandRelationName) {
        URI absoluteUri = client.newURIBuilder(serviceUri).appendEntitySetSegment(entitySetName).appendKeySegment(keyValue)
          .expand(expandRelationName).build();
        return readEntity(edm, absoluteUri);
      }
    
      private ClientEntity readEntity(Edm edm, URI absoluteUri) {
        ODataEntityRequest<ClientEntity> request = client.getRetrieveRequestFactory().getEntityRequest(absoluteUri);
        request.setAccept("application/json;odata.metadata=full");
        ODataRetrieveResponse<ClientEntity> response = request.execute(); 
          
        return response.getBody();
      }
      
      private ClientEntity loadEntity(String path) throws ODataDeserializerException {
        InputStream input = getClass().getResourceAsStream(path);
        return client.getBinder().getODataEntity(client.getDeserializer(ContentType.APPLICATION_JSON).toEntity(input));
      }
    
      public ClientEntity createEntity(Edm edm, String serviceUri, String entitySetName, ClientEntity ce) {
        URI absoluteUri = client.newURIBuilder(serviceUri).appendEntitySetSegment(entitySetName).build();
        return createEntity(edm, absoluteUri, ce);
      }
    
      private ClientEntity createEntity(Edm edm, URI absoluteUri, ClientEntity ce) {
        ODataEntityCreateRequest<ClientEntity> request = client.getCUDRequestFactory()
          .getEntityCreateRequest(absoluteUri, ce);
        request.setAccept("application/json");
        ODataEntityCreateResponse<ClientEntity> response = request.execute(); 
          
        return response.getBody();
      }
    
      public int updateEntity(Edm edm, String serviceUri, String entityName, Object keyValue, ClientEntity ce) {
        URI absoluteUri = client.newURIBuilder(serviceUri).appendEntitySetSegment(entityName)
          .appendKeySegment(keyValue).build();
        ODataEntityUpdateRequest<ClientEntity> request = 
          client.getCUDRequestFactory().getEntityUpdateRequest(absoluteUri, UpdateType.PATCH, ce);
        request.setAccept("application/json;odata.metadata=minimal");
        ODataEntityUpdateResponse<ClientEntity> response = request.execute();
        return response.getStatusCode();
      }
    
      public int deleteEntity(String serviceUri, String entityName, Object keyValue) throws IOException {
        URI absoluteUri = client.newURIBuilder(serviceUri).appendEntitySetSegment(entityName)
          .appendKeySegment(keyValue).build();
        ODataDeleteRequest request = client.getCUDRequestFactory().getDeleteRequest(absoluteUri);
        request.setAccept("application/json;odata.metadata=minimal");
        ODataDeleteResponse response = request.execute();
        return response.getStatusCode();
      }
    }


  [1]: https://gitbox.apache.org/repos/asf?p=olingo-odata4.git
  [2]: https://olingo.apache.org/doc/odata4/tutorials/prerequisites/prerequisites.html