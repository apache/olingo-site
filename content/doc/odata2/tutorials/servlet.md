Title: Servlet Support

# Servlet Support

Originally the Olingo OData 2.0 server library has been implemented
based on JAX-RS to take advantage of the REST based CXF features.

With the development of Olingo URI parser it was possible now also
to offer a purely servlet-based approach. This has some advantages
on restricted runtime environments and reduces the amount of
required dependencies. Disadvantage is that some REST features
which are out of the box are not available.

Both solutions JAX-RS and Non-JAX-RS are supported.

For testing the feature it is recommended to start with the
[sample setup](/doc/odata2/sample-setup.html) using the Olingo archetype and do the following 
modifications.

### Maven Dependencies

For a plain servlet based approach it is possible to exclude JAX-RS
dependencies. A simple way is to exclude them with Maven:

    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-lib</artifactId>
      <version>2.0.4</version>
      <exclusions>
        <exclusion>
          <groupId>javax.ws.rs</groupId>
          <artifactId>javax.ws.rs-api</artifactId>
        </exclusion>
      </exclusions>
    </dependency>

A JAX-RS implementation dependency like Apache CXF is not required
for runtime.

### Adopt web.xml

For using the servlet based approach a Servlet must be configured in web.xml file (and probably an old servlet replaced):

    <servlet>
        <servlet-name>ReferenceScenarioServlet</servlet-name>
        <servlet-class>org.apache.olingo.odata2.core.servlet.ODataServlet</servlet-class>
        <init-param>
            <param-name>org.apache.olingo.odata2.service.factory</param-name>
            <param-value>org.apache.olingo.odata2.ref.processor.ScenarioServiceFactory</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>ReferenceScenarioServlet</servlet-name>
        <url-pattern>/ReferenceScenario.svc/*</url-pattern>
    </servlet-mapping>


Ensure that all runtime dependencies to Apache CXF are removed from the sample project.

After this modification the service should work like before.
