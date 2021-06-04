Title: Delta Token Support

## Delta Token Support 

Delta Query retrieves the changes done to a service. It is supported in OData JPA Processor Library from version 1.4.0. The data returned by the feed for the query would look exactly like any other dataset for an OData query but with delta link at the end. 

There are two approaches you can follow to generate delta token.


### Generating Delta Token
##### First Approach

a. Create a Java class by extending `ODataJPATombstoneEntityListener.java`.

b. Implement the abstract method `getQuery`. In the method, implement the logic to generate a `javax.persistence.Query`. The Query object can be created as shown below in the code snippet. 
		
         if (!resultsView.getStartEntitySet().getName().equals(resultsView.getTargetEntitySet().getName())) {    
		   contextType = JPQLContextType.JOIN;
		 } else {    
		   contextType = JPQLContextType.SELECT;  
		 }   
		 
		 JPQLContext jpqlContext = JPQLContext.createBuilder(contextType, resultsView).build();  
		 JPQLStatement jpqlStatement = JPQLStatement.createBuilder(jpqlContext).build();   
		 
		 Query query = em.createQuery(jpqlStatement);
		 
  The `JPQLStatement` is generated based on the OData request (resultsView). The generated `JPQLStatement` can be enhanced to introduce conditions that filters and fetches delta JPA Entities. To enhance the `JPQLStatement` and add a condition to the WHERE Clause, refer to the following code snippet.

  *Note*: It is up to the JPA application developers to come up with a logic suitable for their use case.

		String deltaToken = ODataJPATombstoneContext.getDeltaToken(); 
		
		Query query = null;
		if (deltaToken != null) {  
		  String statement = jpqlStatement.toString();  
		  String[] statementParts = statement.split(JPQLStatement.KEYWORD.WHERE);  
		  String deltaCondition = jpqlContext.getJPAEntityAlias() + ".creationDate >= {ts '" + deltaToken + "'}";  
		  if (statementParts.length > 1)  
		  {    
		  statement = statementParts[0] + JPQLStatement.DELIMITER.SPACE + JPQLStatement.KEYWORD.WHERE + JPQLStatement.DELIMITER.SPACE + deltaCondition + JPQLStatement.DELIMITER.SPACE + JPQLStatement.Operator.AND + statementParts[1];  
		  }  
		  else {    
		    statement = statementParts[0] + JPQLStatement.DELIMITER.SPACE + JPQLStatement.KEYWORD.WHERE + JPQLStatement.DELIMITER.SPACE + deltaCondition;  
		  }   
		  
		  query = em.createQuery(statement);
		 }
		 else  
		   query = em.createQuery(jpqlStatement.toString()); 

c. Implement the method `generateDeltaToken` to generate a string representation of the delta token. The delta token generated shall be used by the client applications to fetch delta in their subsequent OData requests.
		    
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.000");;    
			Date date = new Date(System.currentTimeMillis());    
			dateFormat.format(date);    
			return dateFormat.format(date);

d. Register the class (created above) as an Entity Listener in JPA Entity.
		
        @Entity
		@Table(name = "T_SALESORDERHEADER")
		@EntityListeners(com.sap.core.odata.processor.ref.jpa.listners.SalesOrderTombstoneListner.class)
		public class SalesOrderHeader { 


##### Second Approach
 
The OData JPA Processor Library also provides support so that the JPA application developer can write a call back method in the Entity Listener class and annotate them with `javax.persistence.PostLoad`. Such call back methods are executed by JPA providers in a stateless manner for every record fetched from the database table. In the call back method, logic can be provided to decide whether a JPA entity fetched from the database table is a delta or not as shown below:

		@PostLoad  
		public void handleDelta(Object entity) {    
		  SalesOrderHeader so = (SalesOrderHeader) entity;    
		  if (so.getCreationDate().getTime() < ODataJPATombstoneContext.getDeltaTokenUTCTimeStamp())      
		    return;    
		  else      
		    addToDelta(entity,ENTITY_NAME);  
		} 













