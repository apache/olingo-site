Title: Adding Function Imports to OData Services with the JPA Processor

# Adding Function Imports to OData Services with the JPA Processor

---


Function imports are used to perform custom operations on a JPA entity in addition to CRUD operations. For example, consider a scenario where you would like to check the availability of an item to promise on the sales order line items. ATP check is a custom operation that can be exposed as a function import in the schema of OData service.

OData JPA Processor Library is enhanced to:

  - Enable Custom Operations as Function Imports

  - Add non JPA entity types as Complex Types to the EDM and use the same as Function Imports Return Type


### Enabling Custom Operations as Function Imports  


1. Create a dependency to EDM Annotation Project. This is required to use the annotations that are defined in the project.

    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-api-annotation</artifactId>
      <version>x.x.x</version>
      <scope>provided</scope>
    </dependency>

2. Create a Java class and annotate the Java methods implementing custom operations with Function Import and Parameter Java annotations as shown below. Java methods can be created in JPA entity types and these methods can be annotated with EDM annotations for function import.

       
        package org.apache.olingo.odata2.jpa.processor.ref.extension;

		import java.util.List;

		import javax.persistence.EntityManager;
		import javax.persistence.Persistence;
		import javax.persistence.Query;

		import org.apache.olingo.odata2.api.annotation.edm.EdmFacets;
		import org.apache.olingo.odata2.api.annotation.edm.EdmFunctionImport;
		import org.apache.olingo.odata2.api.annotation.edm.EdmFunctionImport.HttpMethod;
		import org.apache.olingo.odata2.api.annotation.edm.EdmFunctionImport.ReturnType;
		import org.apache.olingo.odata2.api.annotation.edm.EdmFunctionImport.ReturnType.Type;
		import org.apache.olingo.odata2.api.annotation.edm.EdmFunctionImportParameter;
		import org.apache.olingo.odata2.api.exception.ODataException;
		import org.apache.olingo.odata2.jpa.processor.ref.model.Address;
		import org.apache.olingo.odata2.jpa.processor.ref.model.SalesOrderHeader;
		import org.apache.olingo.odata2.jpa.processor.ref.model.SalesOrderItem;

		public class SalesOrderHeaderProcessor {

		  private EntityManager em;

		  public SalesOrderHeaderProcessor() {
		    em = Persistence.createEntityManagerFactory("salesorderprocessing")
		       .createEntityManager();
		}

		@SuppressWarnings("unchecked")
		@EdmFunctionImport(name = "FindAllSalesOrders", entitySet = "SalesOrders", returnType = @ReturnType(
            type = Type.ENTITY, isCollection = true))
		public List<SalesOrderHeader> findAllSalesOrders(
			@EdmFunctionImportParameter(name = "DeliveryStatusCode",
			    facets = @EdmFacets(maxLength = 2)) final String status) {

		   Query q = em
			   .createQuery("SELECT E1 from SalesOrderHeader E1 WHERE E1.deliveryStatus = '"
                   + status + "'");
			List<SalesOrderHeader> soList = (List<SalesOrderHeader>) q
			    .getResultList();
			return soList;
	    }

		@EdmFunctionImport(name = "CheckATP", returnType = @ReturnType(type = Type.SIMPLE, isCollection = false),
			httpMethod = HttpMethod.GET)
		public boolean checkATP(
		   @EdmFunctionImportParameter(name = "SoID", facets = @EdmFacets(nullable = false)) final Long soID,
		   @EdmFunctionImportParameter(name = "LiId", facets = @EdmFacets(nullable = false)) final Long lineItemID) {
		if (soID == 2L) {
			    return false;
		      } else {
			    return true;
		      }
	    }

		@EdmFunctionImport(returnType = @ReturnType(type = Type.ENTITY, isCollection = true), entitySet = "SalesOrders")
		public SalesOrderHeader calculateNetAmount(
		    @EdmFunctionImportParameter(name = "SoID", facets = @EdmFacets(nullable = false)) final Long soID)
		    throws ODataException {

		if (soID <= 0L) {
		   throw new ODataException("Invalid SoID");
		}

		Query q = em
            .createQuery("SELECT E1 from SalesOrderHeader E1 WHERE E1.soId = "
                + soID + "l");
		if (q.getResultList().isEmpty()) {
		  return null;
		}
		SalesOrderHeader so = (SalesOrderHeader) q.getResultList().get(0);
		double amount = 0;
		for (SalesOrderItem soi : so.getSalesOrderItem()) {
		  amount = amount
              + (soi.getAmount() * soi.getDiscount() * soi.getQuantity());
		}
		so.setNetAmount(amount);
		return so;
		}

		@SuppressWarnings("unchecked")
		@EdmFunctionImport(returnType = @ReturnType(type = Type.COMPLEX))
		public Address getAddress(
		    @EdmFunctionImportParameter(name = "SoID", facets = @EdmFacets(nullable = false)) final Long soID) {
		  Query q = em
              .createQuery("SELECT E1 from SalesOrderHeader E1 WHERE E1.soId = "
                  + soID + "l");
		  List<SalesOrderHeader> soList = (List<SalesOrderHeader>) q
              .getResultList();
		  if (!soList.isEmpty()) {
		  return soList.get(0).getCustomer().getAddress();
		  } else {
		    return null;
		  }
		}

		@EdmFunctionImport(returnType = @ReturnType(type = Type.COMPLEX))
		public OrderValue orderValue(
		    @EdmFunctionImportParameter(name = "SoId", facets = @EdmFacets(nullable = false)) final Long soID) {
		Query q = em
            .createQuery("SELECT E1 from SalesOrderHeader E1 WHERE E1.soId = "
                + soID + "l");
		if (q.getResultList().isEmpty()) {
		  return null;
		}
		SalesOrderHeader so = (SalesOrderHeader) q.getResultList().get(0);
		double amount = 0;
		for (SalesOrderItem soi : so.getSalesOrderItem()) {
		  amount = amount
               + (soi.getAmount() * soi.getDiscount() * soi.getQuantity());
		}
		OrderValue orderValue = new OrderValue();
		orderValue.setAmount(amount);
		orderValue.setCurrency(so.getCurrencyCode());
		return orderValue;
		}

          }

3. Create a Java class by implementing the interface *org.apache.olingo.odata2.jpa.processor.api.model* to register the annotated Java methods.

```java
    public class SalesOrderProcessingExtension implements JPAEdmExtension {
      @Override
      public void extendJPAEdmSchema(final JPAEdmSchemaView arg0 {
        // TODO Auto-generated method stub
      }

      @Override
      public void extendWithOperation(final JPAEdmSchemaView view) {
        view.registerOperations(SalesOrderHeaderProcessor.class, null);
      }
    }
```

*Note*: Use the method *extendWithOperation* to register the list of classes and the methods within the class that needs to be exposed as Function Imports. If the second parameter is passed null, then the OData JPA Processor Library would consider all the annotated methods within the class for Function Import. However, you could also restrict the list of methods that needs to be transformed into function imports within a Java class by passing an array of Java method names as the second parameter.

4. Register the class created in step 3 with *ODataJPAContext* as shown below. The registration can be done during the initialization of *ODataJPAContext* in OData JPA Service Factory along with initializing persistence unit name, entity manager factory instance and optional mapping model.

```java
    oDataJPAContext.setJPAEdmExtension((JPAEdmExtension) new SalesOrderProcessingExtension());
```

*Note*: You must register the class because the OData JPA Processor Library should be informed about the list of Java methods that it needs to process in a project. If we do not register, then OData JPA Processor Library should scan all the classes and the methods in the Java project looking for EDM annotations. In order to avoid such overload, it is mandatory to specify the list of Java methods that shall be transformed into function imports in a class.



### Using Non JPA Entities added to the EDM as Function Imports Return Type 

**Prerequisite**

Add non JPA Entity Types as Complex Types to the EDM. See [Extending the EDM Generated from the JPA Models][1] for more information.

*Note*: The Simple Name of the Java class used as the return type in a Function Import and the name of the EDM Complex Type should be same.


Here is an example, you define the operations inside the `SalesOrderHeaderProcessor` class and then register this class inside `JPAEdmExtension` class `extendWithOperation`. 

##### Sample Code

         @EdmFunctionImport(returnType = @ReturnType(type = Type.COMPLEX))
		      public OrderValue orderValue(
			
			     @EdmFunctionImportParameter(name = "SoId", facets = @EdmFacets(nullable = false)) final Long soID) {
				  Query q = em
				
				      .createQuery("SELECT E1 from SalesOrderHeader E1 WHERE E1.soId = "
						+ soID + "l");
				  if (q.getResultList().isEmpty()) {
				  return null;
				  }
				  SalesOrderHeader so = (SalesOrderHeader) q.getResultList().get(0);
				  double amount = 0;
				  for (SalesOrderItem soi : so.getSalesOrderItem()) {
					amount = amount
					    + (soi.getAmount() * soi.getDiscount() * soi.getQuantity());
				  }
				  OrderValue orderValue = new OrderValue();
				  orderValue.setAmount(amount);
				  orderValue.setCurrency(so.getCurrencyCode());
				  return orderValue;
                  }


  [1]: /doc/odata2/tutorials/ExtendingtheEDM.html
  