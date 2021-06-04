Title: Delta Responses

# Delta Responses

Delta responses is a feature on top of OData 2.0 for requesting changes. The feature is defined in OData 4.0 and this is a preliminary and lightweight implementation close to the OData 4.0 specification [(see here)](http://docs.oasis-open.org/odata/odata/v4.0/cos01/part1-protocol/odata-v4.0-cos01-part1-protocol.html#_Toc372793707).

Because of delta responses are not defined in OData 2.0 this feature is optional.

Features:

* Delta Links (for Atom and Json)
* Tombstones [RFC6721](http://tools.ietf.org/html/rfc6721) for deleted entries in Atom format
* Deleted Entries in Json as a lightweight implementation of [Delta Responses](http://docs.oasis-open.org/odata/odata-json-format/v4.0/cos01/odata-json-format-v4.0-cos01.html#_Toc372793080) 

### Use Case

A client requests a (paged) feed. A server can add a delta link on the last feed page. Using the delta link returns only changed data of the feed to the client. Changed data are feed entries with changed properties or deleted entries.

### Implementation

A server has to implement the `TombstoneCallback` interface:

    public interface TombstoneCallback extends ODataCallback {
    ...
    }

Basically the implementation of this interface has to carry information about deleted data and the delta link which is realized as custom query option:

    http://host:80/service/Rooms?!deltatoken=1234

Finally the following code has to go into a `ODataSingleProcessor` implementation:

    /**
     * @see EntitySetProcessor
     */
    @Override
    public ODataResponse readEntitySet(final GetEntitySetUriInfo uriInfo, final String contentType) throws ODataException {
   
      [...]   

      Map<String, ODataCallback> callbacks = new HashMap<String, ODataCallback>();
      callbacks.put(TombstoneCallback.CALLBACK_KEY_TOMBSTONE, tombstoneCallback);

      EntityProviderWriteProperties properties = EntityProviderWriteProperties.serviceRoot(new URI(BASE_URI)).callbacks(callbacks).build();

      final ODataResponse response = new JsonEntityProvider().writeFeed(entitySet, roomsData, properties);

      [...]   

	  return response;
	}
	
##### Json Response

This is an example for a Json response:

        [...]
         {
            "__metadata":{
               "id":"http://host:80/service/Rooms('2')",
               "uri":"http://host:80/service/Rooms('2')",
               "type":"RefScenario.Room",
               "etag":"W/\"2\""
            },
            "Id":"2",
            "Name":null,
            "Seats":66,
            "Version":2,
            "nr_Employees":{
               "__deferred":{
                  "uri":"http://host:80/service/Rooms('2')/nr_Employees"
               }
            },
            "nr_Building":{
               "__deferred":{
                  "uri":"http://host:80/service/Rooms('2')/nr_Building"
               }
            }
         },
         {
            "@odata.context":"$metadata#Rooms/$deletedEntity",
            "id":"http://host:80/service/Rooms('3')"
         },
         {
            "@odata.context":"$metadata#Rooms/$deletedEntity",
            "id":"http://host:80/service/Rooms('4')"
         },
         {
            "@odata.context":"$metadata#Rooms/$deletedEntity",
            "id":"http://host:80/service/Rooms('5')"
         },

      ],
      "__delta":"http://host:80/service/Rooms?!deltatoken=1234"
      }
    }
    
##### Atom Response

This is an example of an Atom delta response (tombstones, RFC6721)

    <feed ...>
    
    [..]
    
    <entry m:etag="W/&quot;3&quot;">
        <id>http://host:80/service/Rooms('2')</id>
        <title type="text">Rooms</title>
        <updated>2014-01-14T18:11:06.681+01:00</updated>
        <category term="RefScenario.Room" scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme"/>
        <link href="Rooms('2')" rel="edit" title="Room"/>
        <link href="Rooms('2')/nr_Employees" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/nr_Employees" title="nr_Employees" type="application/atom+xml;type=feed"/>
        <link href="Rooms('2')/nr_Building" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/nr_Building" title="nr_Building" type="application/atom+xml;type=entry"/>
        <content type="application/xml">
        <m:properties>
            <d:Id>2</d:Id>
            <d:Name>Neu Schwanstein2</d:Name>
            <d:Seats>20</d:Seats>
            <d:Version>3</d:Version>
        </m:properties>
        </content>
    </entry>
    <at:deleted-entry ref="http://host:80/service/Rooms('2')" when="2014-01-14T18:11:06.682+01:00"/>
    <link rel="delta" href="http://host:80/service/Rooms?!deltatoken=1234"/>
    </feed>
