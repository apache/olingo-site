Title: Custom OData JPA Processor

## Custom OData JPA Processor

OData JPA Processor Library along with transforming an existing JPA model as EDM with no or minimalistic coding also processes the OData request and generates the OData response. However, at times it is required for an application using OData JPA Processor Library to perform some pre-processing of requests and post-processing of responses. To enable this in the application, following steps needs to be performed. 

Custom OData JPA Processor is supported from Apache Olingo release 1.1.0 onwards.

a) Write a Custom OData JPA Processor by extending the class `org.apache.olingo.odata2.jpa.processor.api.ODataJPAProcessor`. In the code snippet below, pre-process and post-process are two private methods that can be written to process the request and response. The instance variable (part of ODataJPAProcessor) `jpaProcessor` can be used to process the OData request. The `jpaProcessor` returns the JPA entities after processing the OData request. The instance variable `responseBuilder` can be used for building the OData response from the processed JPA entities.

		public class CustomODataJPAProcessor extends ODataJPAProcessor{
    
		  @Override
          public ODataResponse readEntitySet(final GetEntitySetUriInfo uriParserResultView, final String contentType)
             throws ODataException {
    
           /* Pre Process Step */
           preprocess ( );

           List<Object> jpaEntities = jpaProcessor.process(uriParserResultView);
    
           /* Post Process Step */
           postProcess( );

           ODataResponse oDataResponse =
               responseBuilder.build(uriParserResultView, jpaEntities, contentType);

           return oDataResponse;
          }

         }
b) Write a Custom OData JPA Service Factory. Implement an OData JPA service factory to create an OData service with custom OData JPA Processor. The default service factory `org.apache.olingo.odata2.jpa.processor.api.ODataJPAServiceFactory` part of the library cannot be used. Hence, create a class by extending `org.apache.olingo.odata2.api.ODataServiceFactory`. Follow the steps below to hook an existing flow to a custom OData JPA Processor. Copy the entire code from `ODataJPAServiceFactory` and replace the code as shown below. 

		  ODataSingleProcessor odataJPAProcessor = accessFactory.createODataProcessor(oDataJPAContext);

    with 

		 ODataSingleProcessor odataJPAProcessor = new CustomODataJPAProcessor(oDataJPAContext);