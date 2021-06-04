Title:    Tutorial - Read service with Olingo V4 - Maven appendix

# Appendix for Maven users

Prerequisites for this Appendix is an installed Maven (Version 3.x) and access to the [Maven Central Repository](www.maven.org)

### Initial project setup

To create a Maven project without Eclipse, execute the following command on command line:

    mvn archetype:generate \
      -DgroupId=org.apache.olingo \
      -DartifactId=DemoService \
      -DarchetypeArtifactId=maven-archetype-webapp \
      -DinteractiveMode=false


This creates a project skeleton.
Afterwards, use the goal `eclipse:eclipse` in order to make the project Eclipse-like.
This allows to import the project into Eclipse using the default Eclipse-importer:
`File->Import->General->Existing projects into Workspace`

### Implementation of the Demo Service

To implement the Demo Service see section [4. Implementation](tutorial_read.html#4-implementation) in the tutorial.

### Append Jetty to pom

To add support for run the Demo Service via Maven and the Jetty plugin add following part into the `pom.xml`:


    <build>
      <finalName>${project.artifactId}</finalName>
      <defaultGoal>package jetty:run</defaultGoal>
      <resources>
        <resource>
          <directory>src/main/version</directory>
          <filtering>true</filtering>
          <targetPath>../${project.build.finalName}/gen</targetPath>
        </resource>
        <resource>
          <directory>src/main/resources</directory>
          <filtering>true</filtering>
        </resource>
        <resource>
          <directory>target/maven-shared-archive-resources</directory>
        </resource>
      </resources>

      <plugins>
        <plugin>
          <groupId>org.mortbay.jetty</groupId>
          <artifactId>jetty-maven-plugin</artifactId>
          <version>8.1.14.v20131031</version>
        </plugin>
      </plugins>
    </build>


Afterwards it is possible to call `mvn` (or `mvn jetty:run`) on the console to build the project and start the Jetty server on <http://localhost:8080>
