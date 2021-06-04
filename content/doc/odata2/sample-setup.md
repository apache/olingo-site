Title: Sample Project Setup

# Sample Project Setup

Olingo has prepared a very simple sample car service that can work as a starting point for implementing a custom OData service.
This service consists of a very simple EDM with two entity sets that are cars and manufactures and a memory based data provider that is a simple hash map.
Therefore the project implements a very basic single OData processor supporting a minimal readonly scenario.
If build with Maven the build result is a web application (`war` file) which can be deployed to any JEE compliant web application server (e.g. [Tomcat](http://tomcat.apache.org)).

---

### Maven Archetype

Apache Olingo supports Maven archetypes that are a kind of project template for setting up new projects from scratch.
Currently exists an archetype with an `ODataSingleProcessor` implementation as `olingo-odata2-sample-cars-service-archetype` and an archetype with an annotation based `ODataService` implementation as `olingo-odata2-sample-cars-annotation-archetype`.

To generate the sample project for the `ODataSingleProcessor` implementation start with:

    mvn archetype:generate \
      -DinteractiveMode=false \
      -Dversion=1.0.0-SNAPSHOT \
      -DgroupId=com.sample \
      -DartifactId=my-car-service \
      -DarchetypeGroupId=org.apache.olingo \
      -DarchetypeArtifactId=olingo-odata2-sample-cars-service-archetype \
      -DarchetypeVersion=RELEASE

To generate the sample project for the `ODataService`  implementation with use of the Java Annotations extension start with:

    mvn archetype:generate \
      -DinteractiveMode=false \
      -Dversion=1.0.0-SNAPSHOT \
      -DgroupId=com.sample \
      -DartifactId=my-car-service \
      -DarchetypeGroupId=org.apache.olingo \
      -DarchetypeArtifactId=olingo-odata2-sample-cars-annotation-archetype \
      -DarchetypeVersion=RELEASE

If an archetype is not available via Maven standard configuration then an additional parameter `-DarchetypeRepository=http://repository.apache.org/snapshots` can solve the issue.

Based on the Olingo project template Maven will generate a new project with the specified GAV*) coordinates: `com.sample:my-car-service:1.0.0-SNAPSHOT`.
GAV coordinates can be freely chosen during generation with the interactive mode. To enable the interactive mode `-DinteractiveMode` must be set to true or omitted (to use Maven default setting of `true`).

The result is a new and ready to build Maven project. Switch to *my-car-service* directory and execute:

    mvn clean install

If a Apache Olingo dependency is not available via Maven standard configuration than adding the Apache Maven Repository (or in case you want to use SNAPSHOTS the Apache Snapshot Repository) into your Maven `settings.xml` or the `pom.xml` of this project can solve the issue.

    …
      <repositories>
        <repository>
          <id>apache.central</id>
          <name>Central Repository</name>
          <url>http://repo.maven.apache.org/maven2</url>
        </repository>

        <repository>
          <id>apache.snapshots</id>
          <name>Apache SNAPSHOT Repository</name>
          <url>https://repository.apache.org/content/repositories/snapshots/</url>
        </repository>
      </repositories>
    …


Maven will build the project with the result **car-service.war** in the Maven *target* directory which can be deployed to any JEE compliant web application server.
To call the deployed and running OData service enter this URI in a browser:

    http://localhost:8080/my-car-service/

Which show a entry page for the generated sample service with links to the *Metadata* (`$metadata`), *Service Document* and some *sample data* which it provides.

*) GAV means a Maven groupId, artifactId and version.

### Eclipse IDE Support

The archetype template supports Eclipse as IDE.
Additionally to a Maven clean and install it is possible to call the following Maven goal:

    mvn eclipse:clean eclipse:eclipse

This will generate Eclipse project files including all transitive dependencies and the web application facet.
Import the project to Eclipse and it should be recognized as a web application project.
Deploy the Eclipse project to a server and it should run as well.
