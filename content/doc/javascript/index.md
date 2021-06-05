Title: Olingo OData Client for JavaScript

## Olingo OData Client for JavaScript
The Olingo OData Client for JavaScript (odatajs) is a library written in JavaScript that enables browser based frontend applications to easily use the OData protocol for communication with application servers.

This library "odatajs-4.0.0.min.js" supports only the OData V4 protocol.

For using the OData protocols V1-V3 please refer to the [datajs library](http://datajs.codeplex.com/)

The odatajs library can be included in any html page with the script tag (for example)
```
<script type="text/javascript" src="./sources/odatajs-4.0.0.min.js"></script>
```
and its features can be used through the `odatajs` namespace (or `window.odatajs`). The odatajs library can be used together with the datajs library which uses the `window.OData` namespace.

For API documentation please see [ODatajs API documentation](/doc/javascript/apidoc/index.html)

You may also use the documentation and the samples from the [datajs library](http://datajs.codeplex.com/documentation) because the features and API are similar.

## Features

Features ported from DataJS V3 to Olingo OData Client for JavaScript to support OData V4

* Support of OData V4 headers
* Support of OData V4 metadata payload
* Support of OData JSON payload version 4.0 (include batch request and response payload)
* Support of the Cache and Store modules
Changes in Olingo OData Client for JavaScript
* [Changed] The license header is changed to the Apache license header
* [Pruned] Atom and JSON verbose payload support is removed to conform the OData V4 protocol
* [New] New build infrastructure - 
In order to make the build infrastructure of ODataJS cross-platform and easy accessible, a new build infrastructure based on Node Package Manager (NPM) and Grunt is adopted and has replaced the former Visual Studio based build infrastructure. For more details, please refers to the building instructions.
* [New] The code comments are modified to the JSDoc style and the JSDoc generation functionality is enabled.
* [New] Support running on browser-independent environments (e.g. Node.js)

## Contribute to Olingo OData Client for JavaScript
If you are interested to contribute to this library please have a look into [Project setup](/doc/javascript/project-setup.html) and [Build instructions](/doc/javascript/project-build.html) where you find a manual how you can download the source code and build the odatajs library.

If you intend so please also join the [Olingo developers group](/support.html) for discussion.