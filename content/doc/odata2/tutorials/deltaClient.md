Title: Consuming Delta Responses

# Consuming Delta Responses

Delta responses is a feature on top of OData 2.0 for requesting changes. The feature is defined in OData 4.0 and this is a preliminary and lightweight implementation close to the OData 4.0 specification [(see here)](http://docs.oasis-open.org/odata/odata/v4.0/errata02/os/complete/part1-protocol/odata-v4.0-errata02-os-part1-protocol-complete.html#_Toc406398316).

How delta responses can be produced by an OData service is documented here: [server side delta responses](/doc/odata2/tutorials/delta.html).

### Use Case 

A client reads a feed and later wants to get only the update of changed and deleted entries. 

### Principle

A client has to read a complete (paged) feed. With the last feed page the service has to provide a delta link instead of a next link. A delta link us usually the same feed url containing a delta token as proprietary query parameter. With the delta link the client can query the service again and gets back a delta feed containing entries which were changed or deleted since the complete feed was received.

![Class Diagram Delta Feeds](/img/DeltaFeedClassDiagram.png)

### Examples

Example for a delta link: `http://<host>:<port>/odata/Rooms?!deltatoken=123`

##### Delta Feed Handling

Depends on the general client sample [here](...)

    // retrieve a feed
    ODataFeed feed = client.readFeed("Container1", "Rooms", contentType);
    String deltaLink = feed.getFeedMetadata().getDeltaLink();

    // store feed data
    List<ODataEntry> entries = feed.getEntries();

    // get delta link from feed
    String deltaLink = feed.getFeedMetadata().getDeltaLink();

    // query delta feed using delta link
    ODataDeltaFeed deltaFeed = client.readDeltaFeed("Container1", "Rooms", contentType, deltaLink);

    List<ODataEntry> changedEntries = deltaFeed.getEntries();
    List<DeletedEntryMetada) deletedEntries = deltaFeed.getDeletedEntries();
    
	// proceed with data handling of entries, changedEntries and deletedEntries    

##### Response Deserialization

Precondition: Query the delta link using any HTTP client.
	
    InputStream content = ...; // retrieve content    
   
    EdmEntityContainer entityContainer = edm.getEntityContainer("Container1");

    ODataFeed deltaFeed = EntityProvider.readDeltaFeed(contentType, entityContainer.getEntitySet("Rooms"), 
      content, EntityProviderReadProperties.init().build());

### Links

* Delta Links (for Atom and Json)
* Tombstones [RFC6721](http://tools.ietf.org/html/rfc6721) for deleted entries in Atom format
* Deleted Entries in Json as a lightweight implementation of [Delta Responses](http://docs.oasis-open.org/odata/odata-json-format/v4.0/cos01/odata-json-format-v4.0-cos01.html#_Toc372793080) 

