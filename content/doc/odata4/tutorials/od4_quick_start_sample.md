Title: A QuickStart Guide with our OData 4.0 Sample service

# A QuickStart Guide with our OData 4.0 Sample service

---
### Overview
Olingo has prepared a very simple sample car service that can work as a starting point for implementing a custom OData service. This service consists of a very simple EDM with two entity sets that are cars and manufactures and a memory based data provider that is a simple hash map. Therefore the project implements a very basic single OData processor supporting a minimal readonly scenario.

This sample can be deployed to any JEE compliant web application server (e.g. [Tomcat](http://tomcat.apache.org)) and be used to get an overview on how to provide an OData V4 compliant webserver. 

### Prerequisites

To run this sample you will need to perfom the following steps:

* Get any JEE compliant web application server (e.g. [Tomcat](http://tomcat.apache.org))
* Get the Sample war file (See the next chapter of this tutorial)
* Deploy the war file to your server

You can now explore the service.

### Get the war file via the maven repository

To get the war file via the maven repository you can simple look [here](https://repository.apache.org/index.html#nexus-search;gav~org.apache.olingo~odata-server-sample```) and download the war file. 

### Build the war file on your own

You can also build the war file yourself using maven. To do this please set up your project by cloning our git repository. You can find a how to [here](/doc/odata4/maven.html)

After you have executed the maven goals "clean install" you can find the war file in the target folder under "olingo-odata4/samples/server/target".

If you would like to debug the service you can do so by integrating the whole project into eclipse by following this [tutorial](/doc/odata4/eclipse.html).



