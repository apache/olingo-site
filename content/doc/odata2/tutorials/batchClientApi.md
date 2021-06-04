Title: Batch Request construction

Batch Request construction
--------------------------

**Query Request construction**

A BatchQueryPart is a representation of a single retrieve request. You can use the following methods in order to fill out a request:

  - method(String) 
  - uri(String) 
  - contentId(String) 
  - headers(List<String>)

<pre><code>
BatchQueryPart request = BatchQueryPart.method("GET").uri("$metadata").build();
</pre></code>

**Note:** The valid method value is GET.

**ChangeSet construction**
A BatchChangeSetPart is a representation of a single change request. You can use the following methods in order to fill out a change request:

  - method(String)
  - uri(String)
  - headers(List<String>)
  - contentId(String)
  - body(String)

<pre><code>Map<String, String> changeSetHeaders = new HashMap<String, String>();
changeSetHeaders.put("content-type", "application/json;odata=verbose");
BatchChangeSetPart changeRequest = BatchChangeSetPart.method("PUT")
.uri("Employees('2')/EmployeeName")
.headers(changeSetHeaders)
.body("{\"EmployeeName\":\"Frederic Fall MODIFIED\"}")
.build();
...
</pre></code>

**Note:** The valid method values are POST, PUT, DELETE or MERGE.

The change request has to become a part of a changeSet. For that you need to create a changeSet object and to attach the change request to this object.

    ...
    BatchChangeSet changeSet = BatchChangeSet.newBuilder().build();
    changeSet.add(changeRequest);

**Batch request payload construction**
After you collected all created parts, you can call the method writeBatchRequestBody(..) provided by EntityProvider

    ...
    List<BatchPart> batchParts = new ArrayList<BatchPart>();
    batchParts.add(request);
    batchParts.add(changeSet);
     
    InputStream payload = EntityProvider.writeBatchRequest(batchParts, BOUNDARY);

The second parameter BOUNDARY is necessary information for the construction of the batch request payload. It is the value of the boundary parameter, that is set in Content-Type header of the batch request.

**Batch Response interpretation**
Interpretation of the batch response payload
You receive a list of single response by calling EntityProvider.parseBatchResponse(..)

    List<BatchSingleResponse> responses = EntityProvider.parseBatchResponse(responseBody, contentType);
    for (BatchSingleResponse response : responses) {
          response.getStatusCode());
          response.getStatusInfo());
          response.getHeader(HttpHeaders.CONTENT_TYPE);
          response.getBody();
          response.getContentId();
    }

