Title: Redefining Metadata

### Redefining Metadata
The OData services created from JPA models using OData JPA Processor Library derives the names for its elements from Java Persistence Entity elements. These derived (default) names can be redefined using JPA EDM mapping models. JPA EDM Mapping model can be used to redefine:
* Schema Namespace Name
* Entity Type Names
* Entity Set Names	
* Property Names
* Navigation Property Names 
* Complex Type Names

The OData JPA Processor Library applies certain naming rules to derive the names for the above OData elements by default. Here are the rules:

1. Schema Namespace Name is derived from Java Persistence Unit Name.
2. Entity Type Names are derived from Java Persistence Entity Type Names.
3. Entity Set Names are derived from EDM Entity Type Names suffixed with character "s".
4. Property Names are derived from Java Persistence Entity Attribute Names. The initial character in the property name is converted to an upper-case character.
5. Navigation Property Names are derived from Java Persistence attribute name representing relationships. The navigation property name is suffixed with the word "Details".
6. Complex Type Names are derived from Java Persistence Embeddable type names.

*Note*: The names generated by applying the above rules can be overridden using JPA EDM Mapping models. JPA EDM mapping model can be maintained as an XML document according to the schema.

#### Steps to Redefine the Metadata

1. Create a JPA EDM Mapping model XML according to the schema given below. In the XML, maintain the mapping only for those elements that needs to be redefined. For example, if JPA Entity Type A's name has to be redefined, then maintain an EDM name for the same.
   Link to [Schema][1].
2. Deploy the JPA EDM Mapping model XML file in the root directory of your web application archive (store it in the same directory as 'WEB-INF').
3. Pass the XML name into *ODataJPAContext*. In the method *initializeODataJPAContext*, pass the name of the XML document as shown below:

		oDataJPAContext.setJPAEdmNameMappingModel(<mapping XML filename with XML extension>);
		
4. Compile, deploy and run the web application in a web server. A sample JPA EDM Mapping Model is provided as an example below:

##### Sample JPA EDM Mapping Model

		  <?xml version="1.0" encoding="UTF-8" ?> 
		- <JPAEDMMappingModel xmlns="http://www.apache.org/olingo/odata2/jpa/processor/api/model/mapping">
          - <PersistenceUnit name="salesorderprocessing">
		      <EDMSchemaNamespace>SalesOrderProcessing</EDMSchemaNamespace> 
			- <JPAEntityTypes>
		      - <JPAEntityType name="SalesOrderHeader">
		          <EDMEntityType>SalesOrder</EDMEntityType> 
		          <EDMEntitySet>SalesOrders</EDMEntitySet> 
		        - <JPAAttributes>
		            <JPAAttribute name="soId">ID</JPAAttribute> 
		            <JPAAttribute name="netAmount">NetAmount</JPAAttribute> 
		            <JPAAttribute name="buyerAddress">BuyerAddressInfo</JPAAttribute> 
		          </JPAAttributes>
		        - <JPARelationships>
		            <JPARelationship name="salesOrderItem">SalesOrderLineItemDetails</JPARelationship> 
		            <JPARelationship name="notes">NotesDetails</JPARelationship> 
		          </JPARelationships>
		        </JPAEntityType>
		      - <JPAEntityType name="SalesOrderItem">
		          <EDMEntityType>SalesOrderLineItem</EDMEntityType> 
		          <EDMEntitySet>SalesOrderLineItems</EDMEntitySet> 
		        - <JPAAttributes>
		            <JPAAttribute name="liId">ID</JPAAttribute> 
		            <JPAAttribute name="soId">SalesOrderID</JPAAttribute> 
		          </JPAAttributes>
		        - <JPARelationships>
		            <JPARelationship name="salesOrderHeader">SalesOrderHeaderDetails</JPARelationship> 
		            <JPARelationship name="materials">MaterialDetails</JPARelationship> 
		          </JPARelationships>
		        </JPAEntityType>
		      </JPAEntityTypes>
		    - <JPAEmbeddableTypes>
		      - <JPAEmbeddableType name="Address">
		          <EDMComplexType>AddressInfo</EDMComplexType> 
		        - <JPAAttributes>
		            <JPAAttribute name="houseNumber">Number</JPAAttribute> 
		            <JPAAttribute name="streetName">Street</JPAAttribute> 
		          </JPAAttributes>
		        </JPAEmbeddableType>
		      </JPAEmbeddableTypes>
		    </PersistenceUnit>
		  </JPAEDMMappingModel>

		 


  [1]: /resources/RedefiningTheMetadataSchema