Title: How to use the Batch Client API in OData V4

# How to use the Batch Client API in OData V4

### Construction of OData Client

    ODataClient odata = ODataClientFactory.getClient();
    odata.getConfiguration().setDefaultPubFormat(ContentType.APPLICATION_JSON);

### Construction of a client entity and create request

    ClientObjectFactory factory = getClient().getObjectFactory();
    final ClientEntity entity = factory.newEntity("OData.Demo.Manufacturer");
    entity.getProperties().add(factory.newPrimitiveProperty("Name", factory.newPrimitiveValueBuilder().buildString("MyCarManufacturer")));
         
    final URI targetURI = getClient().newURIBuilder(serviceUrl).appendEntitySetSegment("Manufacturers").build();
    final ODataEntityCreateRequest<ClientEntity> createRequest = getClient().getCUDRequestFactory().getEntityCreateRequest(targetURI, entity);

### Add a create request to a changeset

    BatchManager payloadManager = getClient().getBatchRequestFactory().getBatchRequest(serviceUrl).payloadManager();
    final ODataChangeset changeset = payloadManager.addChangeset();
     
    changeset.addRequest(createRequest);

### Construction of a query request

    final URI targetURI = getClient().newURIBuilder(serviceUrl).appendEntitySetSegment("Manufacturers").appendKeySegment(1).build();
    final URI uri = isRelative ? URI.create(<ServiceUri>).relativize(targetURI) : targetURI;
     
    ODataEntityRequest<ClientEntity> queryReq = getClient().getRetrieveRequestFactory().getEntityRequest(uri);
    queryReq.setAccept(ContentType.APPLICATION_JSON);

### Add query request to payloadManager

    payload.addRequest(queryReq);

### Fetch the batch response

    final ODataBatchResponse response = payload.getResponse();
     
    final Iterator<ODataBatchResponseItem> responseBodyIter = response.getBody();
    final ODataBatchResponseItem changeSetResponse = responseBodyIter.next();

