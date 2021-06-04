Title: Handling BLOB and CLOB Data Types

## Handling BLOB and CLOB Data Types

JPA entities can have properties that are of type `java.sql.Blob` or `java.sql.Clob`. Internally, JPA entities can instantiate a JPA provider (Eclipse Link or Hibernate or ...) specific implementation
of the above two interfaces and bind them to the properties. To enable write on such properties using OData JPA Processor Library, an additional access modifier is required to be added to the JPA 
entities. Following is the proposal on how OData JPA Processor Library handles `java.sql.Blob` and `java.sql.Clob` during metadata generation and runtime processing. 

### EDM Generation

Based on the JPA entity property type, the pseudocode for generating the EDM is as below.

**For `java.sql.Blob`**:

1. Check if JPA entity property is of type byte[] or of type java.sql.Blob and annotated with @Lob annotation. 
2. If Step 1 is true, then generate an EDM property with type as Edm.Binary

**For `java.sql.Clob`**:

1. Check if JPA entity property is of type `java.sql.Clob` and annotated with @Lob annotation. 
2. If Step 1 is true, then generate an EDM property with type as `Edm.String` (with no max length unless a max length is specified). 

### Runtime Processing

**Prerequisites:**

1. It is mandatory to implement the callback interface `org.apache.olingo.odata2.jpa.processor.api.OnJPAWriteContent`.
2. The implemented interface needs to be registered with the service via the method `setOnJPAWriteContent` part of `ODataJPAServiceFactory`. 

Following is the pseudocode for handling the `java.sql.Blob` and `java.sql.Clob` during runtime.

		/* Callback Implementation */

		public class OnDBWriteContent implements OnJPAWriteContent {

		  @Override
          public Blob getJPABlob(byte[] binaryData) throws ODataJPARuntimeException {
            try {
              return new JDBCBlob(binaryData);
            } catch (SerialException e) {
              ODataJPARuntimeException.throwException(ODataJPARuntimeException.INNER_EXCEPTION, e);
            } catch (SQLException e) {
              ODataJPARuntimeException.throwException(ODataJPARuntimeException.INNER_EXCEPTION, e);
            }
            return null;
          }

          @Override
          public Clob getJPAClob(char[] characterData) throws ODataJPARuntimeException {
            try {
              return new JDBCClob(new String(characterData));
            } catch (SQLException e) {
              ODataJPARuntimeException.throwException(ODataJPARuntimeException.INNER_EXCEPTION, e);
            }
              return null;
            }
          }


		/* Call Back registration in Service Factory */
		public class JPAReferenceServiceFactory extends ODataJPAServiceFactory {

		  public static final OnJPAWriteContent onDBWriteContent = new OnDBWriteContent();

		  @Override
          public ODataJPAContext initializeODataJPAContext()
              throws ODataJPARuntimeException {

			....
			........
			..............

			setOnWriteJPAContent(onDBWriteContent); //Register Call Back

			return oDataJPAContext;
		   }
		 }

OData JPA Processor Library internally does the following:

1. Checks if the JPA entity property is of type `java.sql.Blob` or `java.sql.Clob`.
2. If Step 1 is true, then it invokes the registered callback method `getJPABlob` or `getJPAClob` respectively.
3. If callback method is not defined, it throws an exception. 

*Note*: Step 3 is needed because the OData JPA Processor Library is not bound to any specific implementation of BLOB or CLOB interface. 






