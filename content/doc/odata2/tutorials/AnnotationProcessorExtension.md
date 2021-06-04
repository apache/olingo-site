Title: Annotation Processor Extension

# Creating a Web Application with the Annotation Processor Extension

### Shortcut: Creation via Archetype
As a shortcut it is possible to create a sample project which use the Annotation Processor Extension via a Maven Archetype. 
Therefore Maven must be called as shown below:

    mvn archetype:generate \
      -DinteractiveMode=false \
      -Dversion=1.0.0-SNAPSHOT \
      -DgroupId=com.sample \
      -DartifactId=my-car-service \
      -DarchetypeGroupId=org.apache.olingo \
      -DarchetypeArtifactId=olingo-odata2-sample-cars-annotation-archetype \
      -DarchetypeVersion=2.0.0

In the generated sample project you now can simply run Maven with the default goal (run `mvn` in the shell) which compiles the sources and starts an Jetty web server at `http://localhost:8080`.

For more detailed documentation about Archetypes in Olingo take a look into the [sample setup](/doc/odata2/sample-setup) section.

### Creation from Scratch
A project which use the Annotation Processor Extension consists mainly of the model beans, the `ODataServiceFactory` implementation and the web resources (e.g. `web.xml`). 
In addition we use Maven so that it is necessary to create a `pom.xml` for project build information and dependency resolution.

##### Create Maven Project structure
To start a folder is created (e.g. *annotation-from-scratch*) which contains the Maven project.
Within this the default Maven project structure is used, which looks like:

    ./src/main/java 
    ./src/main/resources 
    ./src/main/webapp 

##### Create  Maven pom.xml
After creation of the project structure the default `pom.xml` for building of an `WAR-File` have to be created.
In addition we need the dependency to all necessary _Apache Olingo artifacts_ and to the used `JAX-RS` implementation which in this sample is `Apache CXF`.

The resulting `pom.xml` then looks like:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
    
  <groupId>org.apache.olingo</groupId>
  <artifactId>cars-annotations-sample</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <name>${project.artifactId}</name>
    
  <packaging>war</packaging>
    
  <properties>
    <!-- Dependency Versions -->
    <version.cxf>2.7.6</version.cxf>
    <version.servlet-api>2.5</version.servlet-api>
    <version.jaxrs-api>2.0-m10</version.jaxrs-api>
    <version.olingo>2.0.0</version.olingo>
  </properties>
    
  <build>
    <finalName>${project.artifactId}</finalName>
    <defaultGoal>clean package</defaultGoal>
  </build>

  <dependencies>
    <!-- Apache Olingo Library dependencies -->
    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-api</artifactId>
      <version>${version.olingo}</version>
    </dependency>
    <dependency>
      <artifactId>olingo-odata2-api-annotation</artifactId>
      <groupId>org.apache.olingo</groupId>
      <type>jar</type>
      <version>${version.olingo}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-core</artifactId>
      <version>${version.olingo}</version>
    </dependency>
    <!-- Apache Olingo Annotation Processor Extension dependencies -->
    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-annotation-processor-api</artifactId>
      <version>${version.olingo}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-annotation-processor-core</artifactId>
      <version>${version.olingo}</version>
    </dependency>
    <!-- Servlet/REST dependencies -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>${version.servlet-api}</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.ws.rs</groupId>
      <artifactId>javax.ws.rs-api</artifactId>
      <version>${version.jaxrs-api}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-rt-frontend-jaxrs</artifactId>
      <version>${version.cxf}</version>
    </dependency>
  </dependencies>
</project>
```

##### Create Sample (Entity) Model

For this sample a simple model with the two entities _Manufacturer_ and _Car_ is created.

The _Manufacturer_ consists of an `Id`, `Name`, `Founded` and a relation to a list of its `Cars`.  
The _Car_ consists of an `Id`, `Model`, `ProductionYear`, `Price` and a relation to its `Manufacturer`.

**Create Java Beans for Entities**
For each of both entities first a java bean (_POJO_) is created in the package `org.apache.olingo.sample.annotation.model` (which results in a created folder `src/main/java/org/apache/olingo/sample/annotation/model/`) which looks like:

```java
package org.apache.olingo.sample.annotation.model;

/** required Imports */

public class Manufacturer {
  private String id;
  private String name;
  private Calendar founded;
  private List<Car> cars = new ArrayList<Car>();
  
  /** optional getter and setter */
}
```

and:

```java
package org.apache.olingo.sample.annotation.model;
    
/** required Imports */
    
public class Car {
  private String id;
  private String model;
  private Double price;
  private Integer productionYear;
  private Manufacturer manufacturer;
  
  /** optional getter and setter */
}
```

**Annotated created Java Beans**
Now those beans have to be annotated with the annotations of the _Annotation Processor Extension_.

Both beans needs at first the `@EdmEntityType` and `@EdmEntitySet` annotation to define that they represent an OData Entity. These annotation must be added at the bean class which as example for the _Manufacturer_ then look like:

```java
@EdmEntityType
@EdmEntitySet
public class Manufacturer { /** more code */ }
```

Then all simple properties of the Entity must be annotated with `@EdmProperty`, the _Key_ for the Entity additional must be annotated with `@EdmKey` which is in this sample the `Id` field of the entities.

For the _Manufacturer_ it then look like:

```java
@EdmEntityType
@EdmEntitySet
public class Manufacturer {
  @EdmKey
  @EdmProperty
  private String id;
  @EdmProperty
  private String name;
  @EdmProperty
  private Calendar founded;

 /** more code */ 
}
```

A relation to another Entity must be annotated with `@EdmNavigationProperty`. In this sample this are the bi-directional relation between a _Manufacturer_ and its _Cars_.  

For the _Manufacturer_ the added annotation look like:

```java
@EdmEntityType
@EdmEntitySet
public class Manufacturer {
  /** more code */ 

  @EdmNavigationProperty
  private List<Car> cars = new ArrayList<Car>();
    
  /** more code */ 
}
```

The complete resulting Entities (POJOs) then look like:

```java
package org.apache.olingo.sample.annotation.model;
    
import java.util.*;
import org.apache.olingo.odata2.api.annotation.edm.*;
    
@EdmEntityType
@EdmEntitySet
public class Manufacturer {
  @EdmKey
  @EdmProperty
  private String id;
  @EdmProperty
  private String name;
  @EdmProperty
  private Calendar founded;
  @EdmNavigationProperty
  private List<Car> cars = new ArrayList<Car>();
   
  /** optional getter and setter */
}
```

and

```java
package org.apache.olingo.sample.annotation.model;
    
import org.apache.olingo.odata2.api.annotation.edm.*;
    
@EdmEntityType
@EdmEntitySet
public class Car {
  @EdmKey
  @EdmProperty
  private String id;
  @EdmProperty
  private String model;
  @EdmProperty
  private Double price;
  @EdmProperty
  private Integer productionYear;
  @EdmNavigationProperty
  private Manufacturer manufacturer;
    
  /** optional getter and setter */
}
```

The next step is to create the `ODataService`.

##### Create ODataService
The `ODataService` is created via an `ODataServiceFactory` implementation.
For the sample a `AnnotationSampleServiceFactory` in the package `org.apache.olingo.sample.annotation.processor` (which results in a created folder `src/main/java/org/apache/olingo/sample/annotation/processor/`) is created which  extends the `ODataServiceFactory`. The resulting code look like:

```java
package org.apache.olingo.sample.annotation.processor;

/** required Imports */

public class AnnotationSampleServiceFactory extends ODataServiceFactory {
  @Override
  public ODataService createService(final ODataContext context) throws ODataException {
    return null;
  }
}
```

In the `createService(...)` method now the `ODataService` needs to be created.
The _Annotation Processor Extension_ provides therefore the method `createAnnotationService(...)` within the `AnnotationServiceFactory` which can be used. This method require as parameter the _Package_ which contains the _Model_ in form of annotated POJOs (as created in the section _Create the Model_).  

For a persistence between several request it is necessary to hold the created `ODataService` in an static instance. In the sample the [Initialization on demand holder idiom](http://en.wikipedia.org/wiki/Initialization-on-demand_holder_idiom) is used.

As result the implementation look like:

```java
package org.apache.olingo.sample.annotation.processor;

import org.apache.olingo.odata2.api.*;
import org.apache.olingo.odata2.api.exception.*;
import org.apache.olingo.odata2.api.processor.ODataContext;
import org.apache.olingo.odata2.annotation.processor.api.AnnotationServiceFactory;

public class AnnotationSampleServiceFactory extends ODataServiceFactory {

  /**
   * Instance holder for all annotation relevant instances which should be used as singleton
   * instances within the ODataApplication (ODataService)
   */
  private static class AnnotationInstances {
    final static String MODEL_PACKAGE = "org.apache.olingo.sample.annotation.model";
    final static ODataService ANNOTATION_ODATA_SERVICE;
        
    static {
      try {
        ANNOTATION_ODATA_SERVICE = AnnotationServiceFactory.createAnnotationService(MODEL_PACKAGE);
      } catch (ODataApplicationException ex) {
        throw new RuntimeException("Exception during sample data generation.", ex);
      } catch (ODataException ex) {
        throw new RuntimeException("Exception during data source initialization generation.", ex);
      }
    }
  }

  public ODataService createService(final ODataContext context) throws ODataException {
    return AnnotationInstances.ANNOTATION_ODATA_SERVICE;
  }
}
```

Now the model as well as the service creation is done.
The next step is to provide the necessary resources to run the application within an application server. 

##### Create Web Application resources 
To deploy and run the application on an application server it is necessary to provide a `web.xml` which defines the `JAX-RS` entry point which then calls the sample application.

For this sample `Apache CXF` is used (see `<servlet-class>org.apache.cxf.jaxrs.servlet.CXFNonSpringJaxrsServlet</servlet-class>`) which need as parameter the `javax.ws.rs.Application` and the `org.apache.olingo.odata2.service.factory`.

Therefore the `web.xml` is created in the `src/main/webapp/WEB-INF` folder with following content:

```xml
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
        	id="WebApp_ID" version="2.5">
	    <display-name>org.apache.olingo.sample.annotation</display-name>
        	<servlet>
		        <servlet-name>ServiceServlet</servlet-name>
	        	<servlet-class>org.apache.cxf.jaxrs.servlet.CXFNonSpringJaxrsServlet</servlet-class>
        		<init-param>
			      <param-name>javax.ws.rs.Application</param-name>
		              <param-value>org.apache.olingo.odata2.core.rest.app.ODataApplication</param-value>
	        	</init-param>
        		<init-param>
		            <param-name>org.apache.olingo.odata2.service.factory</param-name>
	    		    <param-value>org.apache.olingo.sample.annotation.processor.AnnotationSampleServiceFactory</param-value>
        		</init-param>
		    <load-on-startup>1</load-on-startup>
	    </servlet>
    
	    <servlet-mapping>
    		    <servlet-name>ServiceServlet</servlet-name>
		    <url-pattern>/AnnotationSample.svc/*</url-pattern>
	    </servlet-mapping>
    </web-app>
```

##### Deploy and Run
Build the project with maven via `mvm clean package` and copy the resulting `WAR-File` from the projects `target` folder in the `deploy` folder of the web application server (e.g. a [Tomcat](http://tomcat.apache.org/)).
As example for a default Tomcat 7.x installation `cp $PROJECT_HOME/target/cars-annotations-sample.war $TOMCAT_HOME/webapps`.

After starting the web application server it is possible to request...

  * ...the *Service Document* via the URL: [http://localhost:8080/cars-annotations-sample/AnnotationSample.svc/](http://localhost:8080/cars-annotations-sample/AnnotationSample.svc/)
  * ...the *Metadata* via the URL: [http://localhost:8080/cars-annotations-sample/AnnotationSample.svc/$metadata](http://localhost:8080/cars-annotations-sample/AnnotationSample.svc/$metadata)
  * ...the *Cars* EntitySet via the URL: [http://localhost:8080/cars-annotations-sample/AnnotationSample.svc/CarSet](http://localhost:8080/cars-annotations-sample/AnnotationSample.svc/CarSet)
  * ...the *Manufacturer* EntitySet via the URL: [http://localhost:8080/cars-annotations-sample/AnnotationSample.svc/ManufacturerSet](http://localhost:8080/cars-annotations-sample/AnnotationSample.svc/ManufacturerSet)

Also it is possible to create *Car* and *Manufacturer* Entities via `HTTP POST` requests.

### More detailed look

A more detailed look into the Annotation Processor Extension can be found in the [wiki](https://wiki.apache.org/Olingo/Documentation/AnnotationProcessor).