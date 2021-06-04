Title: Debug Support and Error Handling

# Debug Support and Error Handling

---

### OData Error Conditions

OData exposes error conditions as HTTP responses with error status code (4xx and 5xx) and it is in the responsibility of a client to handle this situations. For more details see
[OData Error Conditions](http://www.odata.org/documentation/odata-version-2-0/operations#ErrorConditions).

In OData the format for error messages is described in [OData-Atom](http://www.odata.org/documentation/odata-version-2-0/atom-format/) and [Odata-JSON](http://www.odata.org/documentation/odata-version-2-0/json-format/). Apache Olingo OData2 has implemented this this format for error message.

### Debug Support

The OData V2 error message format is limited to the HTTP status codes and gives indicators for client errors (status code 4xx) and server errors (5xx). For development and support this is not sufficient because of more technical information is needed for doing a deep error analysis.

Apache Olingo has implemented a feature to return more error information (stack traces, object states, runtime measurements …) within a response to ease the development and support use case.

For uses cases in a production environment this feature is by default off. The following explains how to enable the feature and gives options to a service implementation to switch if on and off e.g. role based or by configuration.

##### DebugCallback

The debug feature can be enabled by the following callback implementation:

	public class MyDebugCallback implements ODataDebugCallback {
	  @Override
	  public boolean isDebugEnabled() {
	  	boolean isDebug = …; // true|configuration|user role check
	    return isDebug;
	  }
	}

##### Register DebugCallback

In your service factory (`ODataServiceFactory`) implement the following method to register the callback:

```java
	public <T extends ODataCallback> T getCallback(final Class<T> callbackInterface) {
	  T callback

	  if (callbackInterface.isAssignableFrom(MyDebugCallback.class)) {
	    callback = (T) new MyDebugCallback();
	  } else {
	    callback = (T) super.getCallback(callbackInterface);
	  }

	  return callback;
	}
```

If this is in place then the url query option odata-debug=json will return detailed error information in json format for each request.

##### Query for Debug Information

** JSON Debug View**

Request url: http://localhost:8080/olingo-odata2-ref-web/ReferenceScenario.svc/?odata-debug=json

Response:

```json
	{
	  "body": "<?xml version='1.0' encoding='utf-8'?><service xml:base=\"http://localhost:8080/olingo-odata2-ref-web/ReferenceScenario.svc/\" xmlns=\"http://www.w3.org/2007/app\" xmlns:atom=\"http://www.w3.org/2005/Atom\"><workspace><atom:title>Default</atom:title><collection href=\"Employees\"><atom:title>Employees</atom:title></collection><collection href=\"Teams\"><atom:title>Teams</atom:title></collection><collection href=\"Rooms\"><atom:title>Rooms</atom:title></collection><collection href=\"Managers\"><atom:title>Managers</atom:title></collection><collection href=\"Buildings\"><atom:title>Buildings</atom:title></collection><collection href=\"Container2.Photos\"><atom:title>Photos</atom:title></collection></workspace></service>",
	  "request": {
	    "method": "GET",
	    "uri": "https://localhost:8080/olingo-odata2-ref-web/ReferenceScenario.svc/?odata-debug=json",
	    "headers": {
	      "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
	      "accept-encoding": "gzip, deflate",
	      "accept-language": "de-de,de;q=0.8,en-us;q=0.5,en;q=0.3",
	      "connection": "keep-alive",
	      "Content-Type": null,
	      "cookie": "JSESSIONID=C6A2403F354B61B1E645744FABCB7FB6C6BC5DA41BC841823647CF2DFF001556; BIGipServerlocalhost=3543090186.26911.0000",
	      "host": "localhost:8080",
	      "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:24.0) Gecko/20100101 Firefox/24.0",
	      "x-forwarded-for": "173.12.210.11"
	    }
	  },
	  "response": {
	    "status": {
	      "code": 200,
	      "info": "OK"
	    },
	    "headers": {
	      "DataServiceVersion": "1.0",
	      "Content-Type": "application/xml;charset=utf-8"
	    }
	  },
	  "runtime": [{
	      "class": "ODataRequestHandler",
	      "method": "handle",
	      "duration": 1911,
	      "memory": 191,
	      "children": [
	        {
	          "class": "UriParserImpl",
	          "method": "parse",
	          "duration": 66,
	          "memory": 0,
	          "children": {}
	        },
	        {
	          "class": "Dispatcher",
	          "method": "dispatch",
	          "duration": 1029,
	          "memory": 95,
	          "children": {}
	        }
	      ]
	    }]
	}
```

** HTML Debug View**

Request url: http://localhost:8080/olingo-odata2-ref-web/ReferenceScenario.svc/?odata-debug=html
to get a self-contained HTML document with all information that is in the JSON
output but can be viewed conveniently in a browser.

##### Custom Debug Output

Starting with release 1.2, it is possible to create custom debug output.
The complete formatting of the debug-support information can be implemented
in a callback method.

Add to the already existing `getCallback` method before the line with `else` a condition check whether the given `ODataCallback` is a `DebugWrapperCallback`.

```java
	} else if (callbackInterface.isAssignableFrom(ODataDebugResponseWrapperCallback.class)) {
	  callback = (T) new DebugWrapperCallback();
	}
```

which then results in following method

```java
	public <T extends ODataCallback> T getCallback(final Class<T> callbackInterface) {
	  T callback

	  if (callbackInterface.isAssignableFrom(MyDebugCallback.class)) {
	    callback = (T) new MyDebugCallback();
	  } else if (callbackInterface.isAssignableFrom(ODataDebugResponseWrapperCallback.class)) {
	    callback = (T) new DebugWrapperCallback();
	  } else {
	    callback = (T) super.getCallback(callbackInterface);
	  }

	  return callback;
	}
```

and implement the callback class

```java
	private final class DebugWrapperCallback implements ODataDebugResponseWrapperCallback {
	  @Override
	  public ODataResponse handle(final ODataContext context, final ODataRequest request, final ODataResponse response,
	      final UriInfo uriInfo, final Exception exception) {
	    if ("true".equals(request.getQueryParameters().get("my-debug"))) {
	      return DebugResponseWrapper.handle(context, request, response, uriInfo, exception);
	    } else {
	      return response;
	    }
	  }
	}
```

where `DebugResponseWrapper` is a class you have to implement which does
the real work.

**Please note** that this callback is not called if the built-in debug output
is requested as described above.

### Log and Trace Support

Apache Olingo has no dependencies to any specific log and trace api (e.g. slf4j, log4j …) and with that it does not trace anything by default. This is to keep the library independent from a specific api so that it can be used on any JEE compliant platform independent from which l&t api is offered there.

Anyhow log and trace is required for most production environments and the following recommendations are given:

##### Servlet Filter

For tracing the http traffic (request url, query parameter, http headers, response code …) to and from the server it is recommended to implement a servlet filter on top of the service. This is completely independent from the Apache Olingo OData library and has no restrictions.

##### Service Processor Implementation

To trace OData activities (read/write activities) at the server it is recommended to do that within a custom processor implementation.

##### Error Callback

Because of OData requires to handle error situations someone can hook into the handling and trace information there.

Simply implement another `ODataCallback` interface and register it within a `ODataServiceFactory`.

Callback:

	public class MyErrorCallback implements ODataErrorCallback {
	  @Override
	  public ODataResponse handleError(ODataErrorContext context) throws ODataApplicationException {
	    LOGGER.severe(context.getException().getClass().getName() + ":" + context.getMessage());
            return EntityProvider.writeErrorDocument(context);
	  }
	}

Register Callback:

	public <T extends ODataCallback> T getCallback(final Class<T> callbackInterface) {
	  T callback

	  if (callbackInterface.isAssignableFrom(MyDebugCallback.class)) {
	    callback = (T) new MyDebugCallback();
	  } else if (callbackInterface.isAssignableFrom(MyErrorCallback.class)) {
	    callback = (T) new MyErrorCallback();
	  } else {
	    callback = (T) super.getCallback(callbackInterface);
	  }

	  return callback;
	}
