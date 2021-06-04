Title: Batch

# Batch

### How to process an OData Batch Request
##### Implement the method executeBatch of the class ODataSingleProcessor.
- Use the method `EntityProvider.parseBatchRequest(contentType, content, batchProperties)` in order to parse the Batch Request Body. As a result you get a list with batch parts. Each part separately represents a ChangeSet or a query operation. 
- Call `handler.handleBathPart(batchRequestPart)` while looping over the list. The handler delegates the processing of a batch part depending on the type of that batch part. 
- When all batch parts are completely processed, use the method `EntityProvider.writeBatchResponse(final List<BatchResponsePart> batchResponseParts)` in order to write all responses in one Batch Response. 
The following example shows a possible implementation:

**Sample Code**

    @Override
    public ODataResponse executeBatch(final BatchHandler handler, final String contentType, final InputStream content)
        throws ODataException {

      List<BatchResponsePart> batchResponseParts = new ArrayList<BatchResponsePart>();
      PathInfo pathInfo = getContext().getPathInfo();
      EntityProviderBatchProperties batchProperties = EntityProviderBatchProperties.init().pathInfo(pathInfo).build();
      List<BatchRequestPart> batchParts = EntityProvider.parseBatchRequest(contentType, content, batchProperties);
      for (BatchRequestPart batchPart : batchParts) {
        batchResponseParts.add(handler.handleBatchPart(batchPart));
      }
      return EntityProvider.writeBatchResponse(batchResponseParts);
    }


**NOTE:** The parameter batchProperties of the method parseBatchRequest contains OData URI informations as PathInfo-object. These informations are necessary for the parsing, that's why the PathInfo-object should not be null.

##### Implement the method executeChangeSet of the class ODataSingleProcessor.
In order to process a request invoke `handler.handleRequest(request)`, that delegates a handling of the request to the request handler and provides ODataResponse. 
Define a rollback semantics that may be applied when a request within a ChangeSet fails. 
The following example shows a possible implementation:

    @Override
     public BatchResponsePart executeChangeSet(final BatchHandler handler, final List<ODataRequest> requests) throws ODataException {
        List<ODataResponse> responses = new ArrayList<ODataResponse>();
        for (ODataRequest request : requests) {
          ODataResponse response = handler.handleRequest(request);
          if (response.getStatus().getStatusCode() >= HttpStatusCodes.BAD_REQUEST.getStatusCode()) {
            // Rollback
            List<ODataResponse> errorResponses = new ArrayList<ODataResponse>(1);
            errorResponses.add(response);
            return BatchResponsePart.responses(errorResponses).changeSet(false).build();
          }
          responses.add(response);
        }
        return BatchResponsePart.responses(responses).changeSet(true).build();
      }

**NOTE:** If a request within a ChangeSet fails, a Batch Response Part contains only the error response and the flag changeSet is set to false.

##### Batch Request Body Example
    --batch_123
    Content-Type: multipart/mixed; boundary=changeset_321
    
    --changeset_321
    Content-Type: application/http
    Content-Transfer-Encoding: binary
    
    PUT Employees('2')/EmployeeName HTTP/1.1
    Content-Length: 100
    DataServiceVersion: 1.0
    Content-Type: application/json;odata=verbose
    MaxDataServiceVersion: 2.0
    
    {"EmployeeName":"Frederic Fall MODIFIED"}
    
    --changeset_321--
    
    --batch_123
    Content-Type: application/http
    Content-Transfer-Encoding: binary
    
    GET Employees('2')/EmployeeName?$format=json HTTP/1.1
    Accept: application/atomsvc+xml;q=0.8, application/json;odata=verbose;q=0.5, */*;q=0.1
    MaxDataServiceVersion: 2.0
    
    
    --batch_123--

**NOTE:**

- A Content-Type header of the Batch Request must specify a content type of "multipart/mixed" and a boundary parameter. 
- Each Batch Part is separated by the boundary string, that must occur at the beginning of a line, following a CRLF. 
- The boundary of the ChangeSet should be different from that used by the Batch. 
##### Request Line of a single request
The following request lines of a single request (e.g. a retrieve request) will be accepted:

- GET http://<scheme>/<service_name>/<resource_path> HTTP/1.1 - the request line with an absolute URI 
- GET <resource_path> HTTP/1.1 - the request line that contains a relative-path reference 
Query Options can optionally follow the Resource Path.

**Note:** An absolute-path reference like /<service_name>/<resource_path> will not be accepted

##### Content-ID
The new entity may be referenced by subsequent requests within the same ChangeSet by referring to the Content-Id value. $<contentIdValue> acts as an alias for the Resource Path of the new entity.
In order to refer the new entity the Request URI must begin with $<contentIdValue>:

    POST Customers HTTP/1.1
    Content-ID: newCustomer
    ...
    PUT $newCustomer/Name HTTP/1.1

**Note:** Requests in different ChangeSets cannot reference one another, even if they are in the same Batch

**Note:** Client are expected to take care of the percent encoding of the special characters from their end, if there are any. 
Also, for batch requests the encoding of the parameters in the URLs in the payload are expected to be taken care by the client.

### References
[http://www.odata.org/documentation/odata-v2-documentation/batch-processing/](http://www.odata.org/documentation/odata-v2-documentation/batch-processing/ "External Link")

    