Title: Client Library

# How to use Apache Olingo as Client Library

This Tutorial shows how to use the Apache Olingo Library for CRUD operations on an OData Service.

Therefore it contains the [Explaining the Client](#start) section which explains how to implement the CRUD operations based on sample code and a [Client Quickstart Guide](#quickstart) which give a step by step guide to create a sample and server to see both together in action.

  * [Client Quickstart Guide](#quickstart): Get a running sample client and server within a few minutes (Requirements at least *Java 6 Runtime* and *Maven 3*)
  * [Explaining the Client](#start): The actual *How To* with explanation and sample code for following use cases: [Read the Metadata](#readmetadata), [Read the Data](#readdata), [Create more Data](#createdata), [Update the Data](#updatedata) and [Delete the Data](#deletedata). After this *How To* the sample code from all samples can be put together and be executed against a Sample Service which is shown in the [Run the Client](#run) section.  

---

<a name="quickstart"></a>

### Client Quickstart Guide
With this Quickstart guide the runable sample client and an sample service is created within a few minutes.
Therefore it just requires an installed *Java 6 Runtime*, *Maven 3* and an internet connection.

  1. Start in some folder (e.g. `Quickstart`) which is called our `$ROOT` in the steps below.
  1. Create sample Client project ([more details](#projectstructure))
    2. Create project folder `ClientSample` (in `$ROOT`) and within create following folder structure: `./src/main/java/org/apache/olingo/sample/client` (e.g. `mkdir -p src/main/java/org/apache/olingo/sample/client/`).
    3. In project folder (`ClientSample`) create file `pom.xml` and copy [this sample pom into](#pom).
    4. In folder `./src/main/java/org/apache/olingo/sample/client` create file `OlingoSampleApp.java` and copy [the whole sample client into](#sampleclient).
    5. In project folder `ClientSample` run `mvn` to build the project.
  1. Create and start sample Service ([more details](#sampleservice))
    1. Go to `$ROOT` and create service project from archetype via

```
  mvn archetype:generate
    -DinteractiveMode=false
    -Dversion=1.0.0-SNAPSHOT
    -DgroupId=com.sample
    -DartifactId=my-car-service
    -DarchetypeGroupId=org.apache.olingo
    -DarchetypeArtifactId=olingo-odata2-sample-cars-annotation-archetype
    -DarchetypeVersion=2.0.0
```

    3. Go into from archetype created folder `my-car-service` and just run `mvn` to start the Sample Service on a *Jetty Web Server* at [http://localhost:8080](http://localhost:8080).
  4. Run `OlingoSampleApp` against sample Service ([more details](#runsample))
    1. Go into sample project folder (`ClientSample`) and execute sample client via `java -cp target/OlingoSampleClient.jar org.apache.olingo.sample.client.OlingoSampleApp`.
    2. Now the result of the *read metadata*, *read/create/update/delete data* calls is printed as well as the corresponding *JSON* requests and response body. Within the following section [Explaining the Client](#start) the therefore implemented client methods are shown and explained.

**Optional**
With running `mvn eclipse:eclipse` within the created sample client and sample server projects the necessary Eclipse project files will be generated so that both projects can easily imported into Eclipse (Eclipse -> *File* -> *Import...* *Import existing projects into workspace*).

---

<a name="start"></a>

### Explaining the Client

In this Tutorial an overview is given on how to create, read, update and delete an entity at an existing OData Service with support of the Apache Olingo Library.

Each section explain how to do an operation and shows example code. For simplicity some additional not direct to Apache Olingo releated code is encapsulated in separate methods and is listed in the *Additional code* section at the end of this tutorial. After finishing this tutorial the listed code samples should compile and run against a local running Olingo Sample Service. Therefore all code samples must be *copy / pasted* into a single Java class.

All listed examples (and URIs in the examples) work together with a local running Olingo Sample Service. For set up of this see the *Preparing* section.

<a name="readedm"></a>

##### Read EDM

For an OData Service the *Entity Data Model (EDM)* defines all *metadata* information about the provided data of the service. This includes all entities with their type, properties and relations, which entities are provided as entity sets and additional functions and operations provided by the OData Service. The *EDM* also have to be provided by the OData Service via a unique URI (e.g. `http://localhost:8080/MyFormula.svc/$metadata`) in the *EDMX* format.

This fact is important because the Apache Olingo library requires the *metadata* for serialization and de-serialization of the data of an entity (e.g. the validation of the data is done against the *EDM* provided *metadata*).
Hence the first step in this sample is to read the whole *EDM* of an OData Service.

**Code sample: Read EDM ($metadata)**

~~~java
public Edm readEdm(String serviceUrl) throws IOException, ODataException {
  InputStream content = execute(serviceUrl + "/" + METADATA, APPLICATION_XML, HTTP_METHOD_GET);
  return EntityProvider.readMetadata(content, false);
}
~~~

As shown in the sample code to read the *EDM* a *HTTP GET* on the corresponding *$metadata* URI of the OData Service. The resulting conent in form of an *EDMX* is received via basic `HttpURLConnection` is pased into the `EntityProvider.readMetadata(InputStream content, boolean validate)` method which de-serialize the *EDMX* into an *EDM* object.

For read and de-serialize of the *EDM* this is all what have to be done and the resulting `EDM` instance than can be used for necessary serialization and de-serialization in combination of *CRUD* operations supported by Apache Olingo library. 

<a name="readdata"></a>

##### Read Entity

For reading entities this sample provides two methods.
First is read of a complete *OData Feed* / *Entity Set* and second is read os a single *Entity*.
In general both are based on *HTTP GET* requests to an URI which gets as response the content as an `InputStream` which then can be de-serialized by Apache Olingo into corresponding objects.
For de-serialization the methods `EntityProvider.readFeed(...)` and `EntityProvider.readEntry(...)` are provided which both requires the same types of parameters.
This is the format of the content as *HTTP Content-Type* constant (e.g. `application/atom+xml` / `application/json`),
the *EdmEntitySet* of the to be de-serialized entity (which can be get via the `Edm`/`EdmEntityContainer` objects),
the content as `InputStream` and
the `EntityProviderReadProperties` to configure the read method.

Hence the code samples looks very similar. The main difference is the creation of the `absolutUri` and the method called at the `EntityProvider` for de-serialization. 

**Read complete Feed**

~~~java
  public ODataFeed readFeed(Edm edm, String serviceUri, String contentType, String entitySetName)
      throws IOException, ODataException {
    EdmEntityContainer entityContainer = edm.getDefaultEntityContainer();
    String absolutUri = createUri(serviceUri, entitySetName, null);
    
    InputStream content = (InputStream) connect(absolutUri, contentType, HTTP_METHOD_GET).getContent();
    return EntityProvider.readFeed(contentType,
        entityContainer.getEntitySet(entitySetName),
        content,
        EntityProviderReadProperties.init().build());
  }
~~~

For read of a complete `ODataFeed` the *HTTP GET* request URI is a *EntitySet*. Via the `connect(...)` method the request is done against the created absolut uri and the responding content is returned as `InputStream`. This `InputStream` than will be de-serialized by the `EntitProvider.readFeed(...)` into an `ODataFeed` object which contains all entities provided by the OData Service as well as `FeedMetadata` like *inline count* and *next link*.

**Read single Entry**

~~~java
  public ODataEntry readEntry(Edm edm, String serviceUri, String contentType, String entitySetName, String keyValue)
      throws IOException, ODataException {
    // working with the default entity container
    EdmEntityContainer entityContainer = edm.getDefaultEntityContainer();
    // create absolute uri based on service uri, entity set name and key property value
    String absolutUri = createUri(serviceUri, entitySetName, keyValue);
    
    InputStream content = execute(absolutUri, contentType, HTTP_METHOD_GET);
    
    return EntityProvider.readEntry(contentType,
        entityContainer.getEntitySet(entitySetName),
        content,
        EntityProviderReadProperties.init().build());
  }
~~~

For read of a single `ODataEntry` the *HTTP GET* request URI is an *Entity* for which a *key value* is required for creation of the absolut uri. Via the `connect(...)` method the request is done against the absolut uri and the responding content is returned as `InputStream`. 
This `InputStream` than will be de-serialized by the `EntitProvider.readEntry(...)` into an `ODataEntry` object which contains the data / properties of the entry in form of a Map as well as `EntryMetadata` (which contains *etag*, *id*, *association uris* and *uri* information), `MediaMetadata` if the entry is a *media resource*, information about the entry *contains inline entries* and the `ExpandSelectTreeNode`.

<a name="createdata"></a>

##### Create

To create an entity a *HTTP POST* on the corresponding entity set URI with the whole entity data as *POST Body* in a supported format (e.g. `atom-xml`, `json`) has to be done. With Apache Olingo the required *POST Body* can be created (serialized) with the `EntityProvider.writeEntry(String contentType, EdmEntitySet entitySet, Map<String, Object> data, EntityProviderWriteProperties properties)` method. This method creates an `ODataResponse` object which contains the content (i.e. the required *POST Body*) as `InputStream` which then can be send to the server. If the entry was created successfully an *HTTP Status: 201 created* will be returned as well as the complete entry.  

For simplicity in the code sample below the prepare and execute of the *POST* and the read of the response is separated (see *Part 1: Post* and *Part 2: Read*). The last code sample part shows the convenience method which is used for the create entry case (i.e. absolut uri creation and call of the shared `writeEntry` method). 
Additional the parts below are only code snippets (and no complete method which is *copy/paste* ready), because the *Create (HTTP POST)* and *Update (HTTP PUT)* are similar the whole sample contains an `writeEntry` method which is called by the `createEntry` and `updateEntry` methods. The code sample for the complete `writeEntry` method is listed in the *Shared code* section. 

**Code sample: Create single Entry**

**Part 1: POST entry**  
    
~~~java
    // Map<String, Object> data // ...is a method parameter
        
    HttpURLConnection connection = initializeConnection(absolutUri, contentType, httpMethod);
    // prepare
    EdmEntityContainer entityContainer = edm.getDefaultEntityContainer();
    EdmEntitySet entitySet = entityContainer.getEntitySet(entitySetName);
    URI rootUri = new URI(entitySetName);    
    EntityProviderWriteProperties properties = EntityProviderWriteProperties.serviceRoot(rootUri).build();
    // serialize data into ODataResponse object
    ODataResponse response = EntityProvider.writeEntry(contentType, entitySet, data, properties);
    // get (http) entity which is for default Olingo implementation an InputStream
    byte[] buffer = streamToArray((InputStream) response.getEntity());
    // do the POST   
    connection.getOutputStream().write(buffer);
~~~

First the `HttpURLConnection` is initialized.
Then the Apache Olingo related objects are prepared in form of the `EdmEntitySet` and `EntityProviderWriteProperties`. With the `EntityProviderWriteProperties` is is possible to configure the serialization of the entity. But for the basic create use case it is sufficient to set the *rootUri* of the service which in the sample is the name of the entity set.
The entity itself is provided via a `Map` which contains the data as *property name* to *value* pairs. As example with *Name* as key and *Max* as value in the Map an entity property of *Name* with the value *Max* (or as `XML`: `<d:Name>Max</d:Name>`) will be serialized in the `ODataResponse` object.
If all is prepared the `EntityProvider.writeEntry(contentType, entitySet, data, properties)` method creates the `ODataResponse` which contains the serialized data as entity in an `InputStream`. 
This `InputStream` can direclty used and send to the OData Service via an *HTTP Post* as *HTTP Body*.
In the code sample the `InputStream` is read into a buffer (for simplicity we only read once) which than is written into the `OutputStream` of the `HttpURLConnection`.
After this the entity is posted (created) and the response can be read.

**Part 2: Read response**  
    
~~~java
    // if a entity is created (via POST request) the response body contains the new created entity
    HttpStatusCodes statusCode = HttpStatusCodes.fromStatusCode(connection.getResponseCode());
    if(statusCode == HttpStatusCodes.CREATED) {
      // get the content as InputStream and de-serialize it into an ODataEntry object
      InputStream content = connection.getInputStream();
      content = logRawContent(httpMethod + " response:\n  ", content, "\n");
      ODataEntry entry = EntityProvider.readEntry(contentType,
          entitySet, content, EntityProviderReadProperties.init().build());
    }
~~~

When the *HTTP Post* is done the response can be read.
Starting with the response status code to validate that the creation of the entity was successfull, which is the case if it is a `HttpStatusCodes.CREATED`.
Then get the content as `InputStream` from the `HttpURLConnection` and pass it to the `EntityProvider.readEntry(...)` method for de-serialization.
The necassary `EdmEntitySet` parameter is the same as for the `EntityProvider.writeEntry(...)` and for the sample the default `EntityProviderReadProperties` can be used.
The result is a de-serialized `ODataEntry` which represents the created entity from the OData Service.
      
**Part 3: Create single Entry method**

~~~java
  public ODataEntry createEntry(Edm edm, String serviceUri, String contentType, 
      String entitySetName, Map<String, Object> data) throws Exception {
    String absolutUri = createUri(serviceUri, entitySetName, null);
    return writeEntity(edm, absolutUri, entitySetName, data, contentType, HTTP_METHOD_POST);
  }
~~~

The *Part 1* and *Part 2* code snippets are copied from the shared `writeEntity(...)` method of the sample which is called for *create* and *update* of entities. Hence the convenience method `createEntry` only creates the `absolutUri` for the entity set and then delegates to the `writeEntity` method with correct set `HTTP_METHOD_POST`.


<a name="updatedata"></a>

##### Update

Update of an entity is similar to the creation of an entity. 
Hence the code sample is not only very similar but also *create* and *update* is based on the `writeEntry(...)` method in this tutorial and the sample. To show this of below all *quoted* text is the same as in the *create* section and only the *not* quoted text is different.

To update an entity a *HTTP PUT*  on the corresponding entity URI with the *whole entity data* as *POST Body* in a supported format (e.g. `atom-xml`, `json`) has to be done. 
With an *HTTP MERGE/PATCH* it is also possible to send only the *to be updated* data as *POST Body* and omitting the unchanged data. But this is (currently) not shown within this sample.

> With Apache Olingo the required *POST Body* can be created (serialized) with the `EntityProvider.writeEntry(String contentType, EdmEntitySet entitySet, Map<String, Object> data, EntityProviderWriteProperties properties)` method. This method creates an `ODataResponse` object which contains the content (i.e. the required *POST Body*) as `InputStream` which then can be send to the server. 

If the entry was updated successfully an *HTTP Status: 204 No content* will be returned.  

For simplicity in the code sample below the prepare and execute of the *PUT/PATCH* and the convenience method which is used for the update entry case are separated (see *Part 1: Put* and *Part 2: Update Entry*).

> Additional the parts below are only code snippets (and no complete method which is *copy/paste* ready), because the *Create (HTTP POST)* and *Update (HTTP PUT)* are similar the whole sample contains an `writeEntry` method which is called by the `createEntry` and `updateEntry` methods. The code sample for the complete `writeEntry` method is listed in the *Shared code* section. 

**Code sample: Create single Entry**

**Part 1: PUT entry**  
    
~~~java
    // Map<String, Object> data // ...is a method parameter
        
    HttpURLConnection connection = initializeConnection(absolutUri, contentType, httpMethod);
    // prepare
    EdmEntityContainer entityContainer = edm.getDefaultEntityContainer();
    EdmEntitySet entitySet = entityContainer.getEntitySet(entitySetName);
    URI rootUri = new URI(entitySetName);    
    EntityProviderWriteProperties properties = EntityProviderWriteProperties.serviceRoot(rootUri).build();
    // serialize data into ODataResponse object
    ODataResponse response = EntityProvider.writeEntry(contentType, entitySet, data, properties);
    // get (http) entity which is for default Olingo implementation an InputStream
    byte[] buffer = streamToArray((InputStream) response.getEntity());
    // do the PUT   
    connection.getOutputStream().write(buffer);
    //
    HttpStatusCodes statusCode = HttpStatusCodes.fromStatusCode(connection.getResponseCode());
~~~

<!-- Copy pasted from POST/create -->

> First the `HttpURLConnection` is initialized.
> Then the Apache Olingo related objects are prepared in form of the `EdmEntitySet` and `EntityProviderWriteProperties`. With the `EntityProviderWriteProperties` is is possible to configure the serialization of the entity. But for the basic create use case it is sufficient to set the *rootUri* of the service which in the sample is the name of the entity set.
> The entity itself is provided via a `Map` which contains the data as *property name* to *value* pairs. As example with *Name* as key and *Max* as value in the Map an entity property of *Name* with the value *Max* (or as `XML`: `<d:Name>Max</d:Name>`) will be serialized in the `ODataResponse` object.
If all is prepared the `EntityProvider.writeEntry(contentType, entitySet, data, properties)` method creates the `ODataResponse` which contains the serialized data as entity in an `InputStream`. 
> This `InputStream` can direclty used and send to the OData Service via an *HTTP Post* as *HTTP Body*.
> In the code sample the `InputStream` is read into a buffer (for simplicity we only read once) which than is written into the `OutputStream` of the `HttpURLConnection`.

<!-- End of Copy pasted from POST/create -->

After this the entity is update and the response status code can be checked.

**Part 2: Update Entry method**

~~~java
  public void updateEntry(Edm edm, String serviceUri, String contentType, String entitySetName, 
      String id, Map<String, Object> data) throws Exception {
    String absolutUri = createUri(serviceUri, entitySetName, id);
    writeEntity(edm, absolutUri, entitySetName, data, contentType, HTTP_METHOD_PUT);
  }
~~~

The *Part 1* is copied from the shared `writeEntity(...)` method of the sample which is called for *create* and *update* of entities. Hence the convenience method `createEntry` in *Part 2* only creates the `absolutUri` for the entity and then delegates to the `writeEntity` method with correct set `HTTP_METHOD_PUT`.

<a name="deletedata"></a>

##### Delete

For deletion of an entry just an *HTTP DELETE* request is necessary on the URI of the entity. Hence the Apache Olingo is not necessary to serialize or de-serialize anything for this use case.

~~~java
  public HttpStatusCodes deleteEntry(String serviceUri, String entityName, String id) throws IOException {
    String absolutUri = createUri(serviceUri, entityName, id);
    HttpURLConnection connection = connect(absolutUri, APPLICATION_XML, HTTP_METHOD_DELETE);
    return HttpStatusCodes.fromStatusCode(connection.getResponseCode());
  }
~~~      

So the code for delete of an entry the *HTTP DELETE* request URI is an *Entity* for which a *key value* is required for creation of the absolut uri. Via the `connect(...)` method the request is done against the absolut uri and the responding http status code is returned, which is, if the entry was deleted successfully, an *HTTP Status: 204 No content*.
  

##### Shared Methods

In this section are all methods shared and used from the more Apache Olingo releated methods listed as code samples in the concrete *read*, *create*, *update* and *delete* sections above.

**Code sample: Write Entry for Create/Update**

The `writeEntity(...)` method is used for *creation* and *update* of an entry. Simplified it executes the given `httpMethod` on the given `absolutUri` for the serialized entry based on given `contentType`, `entitySetName` and `data`. Additional for the case of an successfull *create* the `ODataEntry` is returned otherwise `null` is returned.

~~~java
  private ODataEntry writeEntity(Edm edm, String absolutUri, String entitySetName, 
      Map<String, Object> data, String contentType, String httpMethod) 
      throws EdmException, MalformedURLException, IOException, EntityProviderException, URISyntaxException {
    
    HttpURLConnection connection = initializeConnection(absolutUri, contentType, httpMethod);
    
    EdmEntityContainer entityContainer = edm.getDefaultEntityContainer();
    EdmEntitySet entitySet = entityContainer.getEntitySet(entitySetName);
    URI rootUri = new URI(entitySetName);
    
    EntityProviderWriteProperties properties = EntityProviderWriteProperties.serviceRoot(rootUri).build();
    // serialize data into ODataResponse object
    ODataResponse response = EntityProvider.writeEntry(contentType, entitySet, data, properties);
    // get (http) entity which is for default Olingo implementation an InputStream
    Object entity = response.getEntity();
    if (entity instanceof InputStream) {
      byte[] buffer = streamToArray((InputStream) entity);
      // just for logging
      String content = new String(buffer);
      print(httpMethod + " request:\n  " + content + "\n");
      //
      connection.getOutputStream().write(buffer);
    }
    
    // if a entity is created (via POST request) the response body contains the new created entity
    ODataEntry entry = null;
    HttpStatusCodes statusCode = HttpStatusCodes.fromStatusCode(connection.getResponseCode());
    if(statusCode == HttpStatusCodes.CREATED) {
      // get the content as InputStream and de-serialize it into an ODataEntry object
      InputStream content = connection.getInputStream();
      content = logRawContent(httpMethod + " response:\n  ", content, "\n");
      entry = EntityProvider.readEntry(contentType,
          entitySet, content, EntityProviderReadProperties.init().build());
    }
    
    //
    connection.disconnect();
        
    return entry;
  }
~~~

**Creation of absolut URI**    

The `createUri(...)` method is used in every convenience method (*read*, *create*, *update* and *delete*) to build the absolut uri based on the `serviceUri`, `entitySetName` and `id` for the case of a single Entry.
    
~~~java
  private String createUri(String serviceUri, String entitySetName, String id) {
    final StringBuilder absolutUri = new StringBuilder(serviceUri).append("/").append(entitySetName);
    if(id != null) {
      absolutUri.append("(").append(id).append(")");
    }
    return absolutUri.toString();
  }
~~~

**Basic HTTP connection initailization and validation**

The `exectute(...)`, `connect(...)` and `initializeConnection(...)` methods are used for initialization of the connection, execute and/or connect and check basic connection success (via `checkStatus(...)` method) to the OData Service and is therefore used in every use case (*read*, *create*, *update* and *delete*).

~~~java
  private InputStream execute(String relativeUri, String contentType, String httpMethod) throws IOException {
    HttpURLConnection connection = initializeConnection(relativeUri, contentType, httpMethod);

    connection.connect();
    checkStatus(connection);

    InputStream content = connection.getInputStream();
    content = logRawContent(httpMethod + " request:\n  ", content, "\n");
    return content;
  }

  private HttpURLConnection connect(String relativeUri, String contentType, String httpMethod) throws IOException {
    HttpURLConnection connection = initializeConnection(relativeUri, contentType, httpMethod);

    connection.connect();
    checkStatus(connection);

    return connection;
  }

  private HttpURLConnection initializeConnection(String absolutUri, String contentType, String httpMethod)
      throws MalformedURLException, IOException {
    URL url = new URL(absolutUri);
    HttpURLConnection connection = (HttpURLConnection) url.openConnection();

    connection.setRequestMethod(httpMethod);
    connection.setRequestProperty(HTTP_HEADER_ACCEPT, contentType);
    if(HTTP_METHOD_POST.equals(httpMethod) || HTTP_METHOD_PUT.equals(httpMethod)) {
      connection.setDoOutput(true);
      connection.setRequestProperty(HTTP_HEADER_CONTENT_TYPE, contentType);
    }

    return connection;
  }

  private HttpStatusCodes checkStatus(HttpURLConnection connection) throws IOException {
    HttpStatusCodes httpStatusCode = HttpStatusCodes.fromStatusCode(connection.getResponseCode());
    if (400 <= httpStatusCode.getStatusCode() && httpStatusCode.getStatusCode() <= 599) {
      throw new RuntimeException("Http Connection failed with status " + httpStatusCode.getStatusCode() + " " + httpStatusCode.toString());
    }
    return httpStatusCode;
  }
~~~

**Logging**

The `logRawContent(...)` (in combination with `streamToArray(...)`) method is simply used for logging the content of the given `InputStream` and re-creation of an readable `InputStream` which is returned.

~~~java
  private InputStream logRawContent(String prefix, InputStream content, String postfix) throws IOException {
    if(PRINT_RAW_CONTENT) {
      byte[] buffer = streamToArray(content);
      content.close();
      
      print(prefix + new String(buffer) + postfix);

      return new ByteArrayInputStream(buffer);
    }
    return content;
  }
    
  private byte[] streamToArray(InputStream stream) throws IOException {
    byte[] result = new byte[0];
    byte[] tmp = new byte[8192];
    int readCount = stream.read(tmp);
    while(readCount >= 0) {
      byte[] innerTmp = new byte[result.length + readCount];
      System.arraycopy(result, 0, innerTmp, 0, result.length);
      System.arraycopy(tmp, 0, innerTmp, result.length, readCount);
      result = innerTmp;
      readCount = stream.read(tmp);
    }
    return result;
  }
~~~

<!--
### Extended Operations
-->


<a name="run"></a>

### Run the sample
To show a sample how a client can look like and how it than can be used the code samples can be copied into an `OlingoSampleApp` class. 
And to show an use case the `main` method below can be added which calls this `OlingoSampleApp` to *read* the *EDM*, *read* a *ODataFeed* and existing *ODataEntry*, *create* a new *ODataEntry*, *update* this *ODataEntry* and at last *delete* the *ODataEntry*. 
Between the interaction with the `OlingoSampleApp` the results are printend via the `print(...)` and `prettyPrint(...)`' methods.

<a name="together"></a>

##### Put the parts together

**Shortcut**

Instead of copy and paste all code samples from above and the main sample method below in section [Copy and Paste](#copypaste) is the whole sample code as `OlingoSampleApp.java` available ([here](#sampleclient)) as well as a `pom.xml` ([here](#pom)) to set up a maven project which all necessary build information. Only requirements are an installed *Java 6 Runtime* and *Maven 3* environment.

**Main sample method**

~~~java
  public static void main(String[] paras) throws Exception {
    OlingoSampleApp app = new OlingoSampleApp();
    
    String serviceUrl = "http://localhost:8080/MyFormula.svc";
    String usedFormat = APPLICATION_JSON; // could also be APPLICATION_ATOM_XML
    
    print("\n----- Generate sample data ------------------");
    app.generateSampleData(serviceUrl);

    print("\n----- Read Edm ------------------------------");
    Edm edm = app.readEdm(serviceUrl);
    print("Read default EntityContainer: " + edm.getDefaultEntityContainer().getName());

    print("\n----- Read Feed ------------------------------");
    ODataFeed feed = app.readFeed(edm, serviceUrl, usedFormat, "Manufacturers");

    print("Read: " + feed.getEntries().size() + " entries: ");
    for (ODataEntry entry : feed.getEntries()) {
      print("##########");
      print("Entry:\n" + prettyPrint(entry));
      print("##########");
    }
    
    print("\n----- Read Entry ------------------------------");
    ODataEntry entry = app.readEntry(edm, serviceUrl, usedFormat, "Manufacturers", "'1'");
    print("Single Entry: " + prettyPrint(entry));
    
    Map<String, Object> data = new HashMap<String, Object>();
    data.put("Id", "123");
    data.put("Name", "MyCarManufacturer");
    data.put("Founded", new Date());
    //
    Map<String, Object> address = new HashMap<String, Object>();
    address.put("Street", "Main");
    address.put("ZipCode", "42421");
    address.put("City", "Fairy City");
    address.put("Country", "FarFarAway");
    data.put("Address", address);
    //
    print("\n----- Create Entry ------------------------------");
    ODataEntry createdEntry = app.createEntry(edm, serviceUrl, usedFormat, "Manufacturers", data);
    print("Created Entry:\n" + prettyPrint(createdEntry));
    
    print("\n----- Update Entry ------------------------------");
    data.put("Name", "MyCarManufacturer Renamed");
    address.put("Street", "Main Street");
    app.updateEntry(edm, serviceUrl, usedFormat, "Manufacturers", "'123'", data);
    ODataEntry updatedEntry = app.readEntry(edm, serviceUrl, usedFormat, "Manufacturers", "'123'");
    print("Updated Entry successfully:\n" + prettyPrint(updatedEntry));
    
    //
    print("\n----- Delete Entry ------------------------------");
    HttpStatusCodes statusCode = app.deleteEntry(serviceUrl, "Manufacturers", "'123'");
    print("Deletion of Entry was successfully: " + statusCode.getStatusCode() + ": " + statusCode.getInfo());
    
    try {
      print("\n----- Verify Delete Entry ------------------------------");
      app.readEntry(edm, serviceUrl, usedFormat, "Manufacturers", "'123'");
    } catch(Exception e) {
      print(e.getMessage());
    }
  }
~~~

**Genration of sample data**

The `generateSampleData` method is used for creation of sample data at the *ODataService* based on the [Apache Olingo archetype sample service](#sampleservice)).

~~~java
public void generateSampleData(String serviceUrl) throws MalformedURLException, IOException {
  String url = serviceUrl.substring(0, serviceUrl.lastIndexOf(SEPARATOR));
  HttpURLConnection connection = initializeConnection(url + INDEX, APPLICATION_FORM, HTTP_METHOD_POST);
  String content = "genSampleData=true";
  connection.getOutputStream().write(content.getBytes());
  print("Generate response: " + checkStatus(connection));
  connection.disconnect();
}
~~~

**Print methods**

The `print` methods are only for simplified print/write logging to some output channel. In the sample it is just the `System.out` stream.

~~~java
  private static void print(String content) {
    System.out.println(content);
  }

  private static String prettyPrint(ODataEntry createdEntry) {
    return prettyPrint(createdEntry.getProperties(), 0);
  }
  
  private static String prettyPrint(Map<String, Object> properties, int level) {
    StringBuilder b = new StringBuilder();
    Set<Entry<String, Object>> entries = properties.entrySet();
    
    for (Entry<String, Object> entry : entries) {
      intend(b, level);
      b.append(entry.getKey()).append(": ");
      Object value = entry.getValue();
      if(value instanceof Map) {
        value = prettyPrint((Map<String, Object>)value, level+1);
        b.append("\n");
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

  private static void intend(StringBuilder builder, int intendLevel) {
    for (int i = 0; i < intendLevel; i++) {
      builder.append("  ");
    }
  }
~~~

<a name="sampleservice"></a>

##### Sample Service

That the sample Clients `main` method generates real output it is necessary that an OData Service with corresponding *Data Model (EDM)* is available at the `service url` defined in the `main` method (which is in the sample `http://localhost:8080/MyFormula.svc`).
This can easily achieved if the Apache Olingo archetype (with id: `olingo-odata2-sample-cars-annotation-archetype`) is used. 
To use the archtype and create this sample project Maven must be called as shown below:

    mvn archetype:generate \
      -DinteractiveMode=false \
      -Dversion=1.0.0-SNAPSHOT \
      -DgroupId=com.sample \
      -DartifactId=my-car-service \
      -DarchetypeGroupId=org.apache.olingo \
      -DarchetypeArtifactId=olingo-odata2-sample-cars-annotation-archetype \
      -DarchetypeVersion=2.0.0

In the generated sample project you now can simply run Maven with the default goal (run `mvn` in the shell) which compiles the sources and starts a *Jetty Web Server* at [http://localhost:8080](http://localhost:8080).

For more detailed documentation on how to use this archetype take a look into the documentation about Archetypes in Olingo in the [sample setup](/doc/odata2/sample-setup) section.

<a name="runsample"></a>

##### Run command

Must be run from project root directory after build with Maven (`mvn`).

    java -cp target/OlingoSampleClient.jar org.apache.olingo.sample.client.OlingoSampleApp


<a name="projectstructure"></a>

##### Project structure

This section contains the complete Maven Project `pom.xml` and `OlingoSampleApp` to build the project. 
To use it just copy both blocks into files in following directory structure:

    ProjectFolder
    \-pom.xml
    \-src/main/java/org/apache/olingo/sample/client/OlingoSampleApp.java

Then run `mvn` to build the project.
After successfull build the sample client (i.e. its `main`) can be executed via `java -cp target/OlingoSampleClient.jar org.apache.olingo.sample.client.OlingoSampleApp`.
If the corresponding OData Service based on the *archetypeArtifactId=olingo-odata2-sample-cars-annotation-archetype* (see Olingo Annotation Archetype) is running on [http://localhost:8080/MyFormula.svc](http://localhost:8080/MyFormula.svc) the sample work and log some information to the console.

<a name="copypaste"></a>

##### Copy and Paste

<a name="pom"></a>

##### Project Pom

Maven project POM (`pom.xml`)

~~~xml
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
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.apache.olingo</groupId>
  <artifactId>OlingoSampleClient</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>jar</packaging>
  <name>OlingoSampleClient</name>

  <properties>
    <!-- Project dependency versions -->
    <version.olingo>2.0.0</version.olingo>
    <!-- Project plugin versions -->
    <version.eclipse-plugin>2.6</version.eclipse-plugin>
    <version.shade-plugin>2.2</version.shade-plugin>
    <!-- Project build settings -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.build.source>1.6</project.build.source>
  </properties>

  <build>
    <defaultGoal>clean package</defaultGoal>
    <finalName>OlingoSampleClient</finalName>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-eclipse-plugin</artifactId>
        <version>${version.eclipse-plugin}</version>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>${version.shade-plugin}</version>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>

  <dependencies>
    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-api</artifactId>
      <version>${version.olingo}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-core</artifactId>
      <version>${version.olingo}</version>
      <exclusions>
        <exclusion>
          <groupId>javax.ws.rs</groupId>
          <artifactId>javax.ws.rs-api</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
  </dependencies>
</project>
~~~

<a name="sampleclient"></a>

##### Client Sample

Complete `OlingoSampleApp` with `main` which can be *copy/pasted* into a `OlingoSampleApp.java`, then compiled and run.

~~~java
/*******************************************************************************
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
package org.apache.olingo.sample.client;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URI;
import java.net.URISyntaxException;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Set;

import org.apache.olingo.odata2.api.commons.HttpStatusCodes;
import org.apache.olingo.odata2.api.edm.Edm;
import org.apache.olingo.odata2.api.edm.EdmEntityContainer;
import org.apache.olingo.odata2.api.edm.EdmEntitySet;
import org.apache.olingo.odata2.api.edm.EdmException;
import org.apache.olingo.odata2.api.ep.EntityProvider;
import org.apache.olingo.odata2.api.ep.EntityProviderException;
import org.apache.olingo.odata2.api.ep.EntityProviderReadProperties;
import org.apache.olingo.odata2.api.ep.EntityProviderWriteProperties;
import org.apache.olingo.odata2.api.ep.entry.ODataEntry;
import org.apache.olingo.odata2.api.ep.feed.ODataDeltaFeed;
import org.apache.olingo.odata2.api.ep.feed.ODataFeed;
import org.apache.olingo.odata2.api.exception.ODataException;
import org.apache.olingo.odata2.api.processor.ODataResponse;

/**
 *
 */
public class OlingoSampleApp {
  public static final String HTTP_METHOD_PUT = "PUT";
  public static final String HTTP_METHOD_POST = "POST";
  public static final String HTTP_METHOD_GET = "GET";
  private static final String HTTP_METHOD_DELETE = "DELETE";
  
  public static final String HTTP_HEADER_CONTENT_TYPE = "Content-Type";
  public static final String HTTP_HEADER_ACCEPT = "Accept";
  
  public static final String APPLICATION_JSON = "application/json";
  public static final String APPLICATION_XML = "application/xml";
  public static final String APPLICATION_ATOM_XML = "application/atom+xml";
  public static final String APPLICATION_FORM = "application/x-www-form-urlencoded";
  public static final String METADATA = "$metadata";
  public static final String INDEX = "/index.jsp";
  public static final String SEPARATOR = "/";

  public static final boolean PRINT_RAW_CONTENT = true;

  public static void main(String[] paras) throws Exception {
    OlingoSampleApp app = new OlingoSampleApp();
        
    //    String serviceUrl = "http://localhost:8080/cars-annotations-sample/MyFormula.svc";
    String serviceUrl = "http://localhost:8080/MyFormula.svc";
    //    String usedFormat = APPLICATION_ATOM_XML;
    String usedFormat = APPLICATION_JSON;
        
    print("\n----- Generate sample data ------------------------------");
    app.generateSampleData(serviceUrl);

    print("\n----- Read Edm ------------------------------");
    Edm edm = app.readEdm(serviceUrl);
    print("Read default EntityContainer: " + edm.getDefaultEntityContainer().getName());

    print("\n----- Read Feed ------------------------------");
    ODataFeed feed = app.readFeed(edm, serviceUrl, usedFormat, "Manufacturers");

    print("Read: " + feed.getEntries().size() + " entries: ");
    for (ODataEntry entry : feed.getEntries()) {
      print("##########");
      print("Entry:\n" + prettyPrint(entry));
      print("##########");
    }
    
    print("\n----- Read Entry ------------------------------");
    ODataEntry entry = app.readEntry(edm, serviceUrl, usedFormat, "Manufacturers", "'1'");
    print("Single Entry:\n" + prettyPrint(entry));
    
    Map<String, Object> data = new HashMap<String, Object>();
    data.put("Id", "123");
    data.put("Name", "MyCarManufacturer");
    data.put("Founded", new Date());
    //
    Map<String, Object> address = new HashMap<String, Object>();
    address.put("Street", "Main");
    address.put("ZipCode", "42421");
    address.put("City", "Fairy City");
    address.put("Country", "FarFarAway");
    data.put("Address", address);

    //
    print("\n----- Read Entry with $expand  ------------------------------");
    ODataEntry entryExpanded = app.readEntry(edm, serviceUrl, usedFormat, "Manufacturers", "'1'", "Cars");
    print("Single Entry with expanded Cars relation:\n" + prettyPrint(entryExpanded));

    //
    //
    print("\n----- Create Entry ------------------------------");
    ODataEntry createdEntry = app.createEntry(edm, serviceUrl, usedFormat, "Manufacturers", data);
    print("Created Entry:\n" + prettyPrint(createdEntry));
    
    print("\n----- Update Entry ------------------------------");
    data.put("Name", "MyCarManufacturer Renamed");
    address.put("Street", "Main Street");
    app.updateEntry(edm, serviceUrl, usedFormat, "Manufacturers", "'123'", data);
    ODataEntry updatedEntry = app.readEntry(edm, serviceUrl, usedFormat, "Manufacturers", "'123'");
    print("Updated Entry successfully:\n" + prettyPrint(updatedEntry));
    
    //
    print("\n----- Delete Entry ------------------------------");
    HttpStatusCodes statusCode = app.deleteEntry(serviceUrl, "Manufacturers", "'123'");
    print("Deletion of Entry was successfully: " + statusCode.getStatusCode() + ": " + statusCode.getInfo());
        
    try {
      print("\n----- Verify Delete Entry ------------------------------");
      app.readEntry(edm, serviceUrl, usedFormat, "Manufacturers", "'123'");
    } catch(Exception e) {
      print(e.getMessage());
    }
  }

  private static void print(String content) {
    System.out.println(content);
  }

  private static String prettyPrint(ODataEntry createdEntry) {
    return prettyPrint(createdEntry.getProperties(), 0);
  }
      
  private static String prettyPrint(Map<String, Object> properties, int level) {
    StringBuilder b = new StringBuilder();
    Set<Entry<String, Object>> entries = properties.entrySet();
    
    for (Entry<String, Object> entry : entries) {
      intend(b, level);
      b.append(entry.getKey()).append(": ");
      Object value = entry.getValue();
      if(value instanceof Map) {
        value = prettyPrint((Map<String, Object>)value, level+1);
        b.append(value).append("\n");
      } else if(value instanceof Calendar) {
        Calendar cal = (Calendar) value;
        value = SimpleDateFormat.getInstance().format(cal.getTime());
        b.append(value).append("\n");
      } else if(value instanceof ODataDeltaFeed) {
        ODataDeltaFeed feed = (ODataDeltaFeed) value;
        List<ODataEntry> inlineEntries =  feed.getEntries();
        b.append("{");
        for (ODataEntry oDataEntry : inlineEntries) {
          value = prettyPrint((Map<String, Object>)oDataEntry.getProperties(), level+1);
          b.append("\n[\n").append(value).append("\n],");
        }
        b.deleteCharAt(b.length()-1);
        intend(b, level);
        b.append("}\n");
      } else {
        b.append(value).append("\n");
      }
    }
    // remove last line break
    b.deleteCharAt(b.length()-1);
    return b.toString();
  }

  private static void intend(StringBuilder builder, int intendLevel) {
    for (int i = 0; i < intendLevel; i++) {
      builder.append("  ");
    }
  }

  public void generateSampleData(String serviceUrl) throws MalformedURLException, IOException {
    String url = serviceUrl.substring(0, serviceUrl.lastIndexOf(SEPARATOR));
    HttpURLConnection connection = initializeConnection(url + INDEX, APPLICATION_FORM, HTTP_METHOD_POST);
    String content = "genSampleData=true";
    connection.getOutputStream().write(content.getBytes());
    print("Generate response: " + checkStatus(connection));
    connection.disconnect();
  }

  public Edm readEdm(String serviceUrl) throws IOException, ODataException {
    InputStream content = execute(serviceUrl + SEPARATOR + METADATA, APPLICATION_XML, HTTP_METHOD_GET);
    return EntityProvider.readMetadata(content, false);
  }

  public ODataFeed readFeed(Edm edm, String serviceUri, String contentType, String entitySetName)
      throws IOException, ODataException {
    EdmEntityContainer entityContainer = edm.getDefaultEntityContainer();
    String absolutUri = createUri(serviceUri, entitySetName, null);

    InputStream content = execute(absolutUri, contentType, HTTP_METHOD_GET);
    return EntityProvider.readFeed(contentType,
        entityContainer.getEntitySet(entitySetName),
        content,
        EntityProviderReadProperties.init().build());
  }

  public ODataEntry readEntry(Edm edm, String serviceUri, String contentType, String entitySetName, String keyValue)
      throws IOException, ODataException {
    return readEntry(edm, serviceUri, contentType, entitySetName, keyValue, null);
  }

  public ODataEntry readEntry(Edm edm, String serviceUri, String contentType, 
      String entitySetName, String keyValue, String expandRelationName)
      throws IOException, ODataException {
    // working with the default entity container
    EdmEntityContainer entityContainer = edm.getDefaultEntityContainer();
    // create absolute uri based on service uri, entity set name with its key property value and optional expanded relation name
    String absolutUri = createUri(serviceUri, entitySetName, keyValue, expandRelationName);

    InputStream content = execute(absolutUri, contentType, HTTP_METHOD_GET);

    return EntityProvider.readEntry(contentType,
        entityContainer.getEntitySet(entitySetName),
        content,
        EntityProviderReadProperties.init().build());
  }

  private InputStream logRawContent(String prefix, InputStream content, String postfix) throws IOException {
    if(PRINT_RAW_CONTENT) {
      byte[] buffer = streamToArray(content);
      print(prefix + new String(buffer) + postfix);
      return new ByteArrayInputStream(buffer);
    }
    return content;
  }

  private byte[] streamToArray(InputStream stream) throws IOException {
    byte[] result = new byte[0];
    byte[] tmp = new byte[8192];
    int readCount = stream.read(tmp);
    while(readCount >= 0) {
      byte[] innerTmp = new byte[result.length + readCount];
      System.arraycopy(result, 0, innerTmp, 0, result.length);
      System.arraycopy(tmp, 0, innerTmp, result.length, readCount);
      result = innerTmp;
      readCount = stream.read(tmp);
    }
    stream.close();
    return result;
  }
      
  public ODataEntry createEntry(Edm edm, String serviceUri, String contentType, 
      String entitySetName, Map<String, Object> data) throws Exception {
    String absolutUri = createUri(serviceUri, entitySetName, null);
    return writeEntity(edm, absolutUri, entitySetName, data, contentType, HTTP_METHOD_POST);
  }
      
  public void updateEntry(Edm edm, String serviceUri, String contentType, String entitySetName, 
      String id, Map<String, Object> data) throws Exception {
    String absolutUri = createUri(serviceUri, entitySetName, id);
    writeEntity(edm, absolutUri, entitySetName, data, contentType, HTTP_METHOD_PUT);
  }

  public HttpStatusCodes deleteEntry(String serviceUri, String entityName, String id) throws IOException {
    String absolutUri = createUri(serviceUri, entityName, id);
    HttpURLConnection connection = connect(absolutUri, APPLICATION_XML, HTTP_METHOD_DELETE);
    return HttpStatusCodes.fromStatusCode(connection.getResponseCode());
  }

      
  private ODataEntry writeEntity(Edm edm, String absolutUri, String entitySetName, 
      Map<String, Object> data, String contentType, String httpMethod) 
      throws EdmException, MalformedURLException, IOException, EntityProviderException, URISyntaxException {

    HttpURLConnection connection = initializeConnection(absolutUri, contentType, httpMethod);

    EdmEntityContainer entityContainer = edm.getDefaultEntityContainer();
    EdmEntitySet entitySet = entityContainer.getEntitySet(entitySetName);
    URI rootUri = new URI(entitySetName);

    EntityProviderWriteProperties properties = EntityProviderWriteProperties.serviceRoot(rootUri).build();
    // serialize data into ODataResponse object
    ODataResponse response = EntityProvider.writeEntry(contentType, entitySet, data, properties);
    // get (http) entity which is for default Olingo implementation an InputStream
    Object entity = response.getEntity();
    if (entity instanceof InputStream) {
      byte[] buffer = streamToArray((InputStream) entity);
      // just for logging
      String content = new String(buffer);
      print(httpMethod + " request on uri '" + absolutUri + "' with content:\n  " + content + "\n");
      //
      connection.getOutputStream().write(buffer);
    }

    // if a entity is created (via POST request) the response body contains the new created entity
    ODataEntry entry = null;
    HttpStatusCodes statusCode = HttpStatusCodes.fromStatusCode(connection.getResponseCode());
    if(statusCode == HttpStatusCodes.CREATED) {
      // get the content as InputStream and de-serialize it into an ODataEntry object
      InputStream content = connection.getInputStream();
      content = logRawContent(httpMethod + " request on uri '" + absolutUri + "' with content:\n  ", content, "\n");
      entry = EntityProvider.readEntry(contentType,
          entitySet, content, EntityProviderReadProperties.init().build());
    }

    //
    connection.disconnect();
        
    return entry;
  }

  private HttpStatusCodes checkStatus(HttpURLConnection connection) throws IOException {
    HttpStatusCodes httpStatusCode = HttpStatusCodes.fromStatusCode(connection.getResponseCode());
    if (400 <= httpStatusCode.getStatusCode() && httpStatusCode.getStatusCode() <= 599) {
      throw new RuntimeException("Http Connection failed with status " + httpStatusCode.getStatusCode() + " " + httpStatusCode.toString());
    }
    return httpStatusCode;
  }

  private String createUri(String serviceUri, String entitySetName, String id) {
    return createUri(serviceUri, entitySetName, id, null);
  }
      
  private String createUri(String serviceUri, String entitySetName, String id, String expand) {
    final StringBuilder absolutUri = new StringBuilder(serviceUri).append(SEPARATOR).append(entitySetName);
    if(id != null) {
      absolutUri.append("(").append(id).append(")");
    }
    if(expand != null) {
      absolutUri.append("/?$expand=").append(expand);
    }
    return absolutUri.toString();
  }

  private InputStream execute(String relativeUri, String contentType, String httpMethod) throws IOException {
    HttpURLConnection connection = initializeConnection(relativeUri, contentType, httpMethod);

    connection.connect();
    checkStatus(connection);

    InputStream content = connection.getInputStream();
    content = logRawContent(httpMethod + " request on uri '" + relativeUri + "' with content:\n  ", content, "\n");
    return content;
  }

  private HttpURLConnection connect(String relativeUri, String contentType, String httpMethod) throws IOException {
    HttpURLConnection connection = initializeConnection(relativeUri, contentType, httpMethod);

    connection.connect();
    checkStatus(connection);

    return connection;
  }

  private HttpURLConnection initializeConnection(String absolutUri, String contentType, String httpMethod)
      throws MalformedURLException, IOException {
    URL url = new URL(absolutUri);
    HttpURLConnection connection = (HttpURLConnection) url.openConnection();

    connection.setRequestMethod(httpMethod);
    connection.setRequestProperty(HTTP_HEADER_ACCEPT, contentType);
    if(HTTP_METHOD_POST.equals(httpMethod) || HTTP_METHOD_PUT.equals(httpMethod)) {
      connection.setDoOutput(true);
      connection.setRequestProperty(HTTP_HEADER_CONTENT_TYPE, contentType);
    }

    return connection;
  }
}
~~~