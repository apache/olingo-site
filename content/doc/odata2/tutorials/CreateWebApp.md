Title: VCreating a Web Application Project

## Creating a Web Application Project for Transforming JPA Models into OData Services 

In this section, information on how to create a web application (Maven) for transforming JPA Models into OData Services using OData JPA Processor Library is provided.
The table gives the list of Maven dependencies you need to include in the POM.xml of your application.

*Note*: The following dependencies are applicable for an application using EclipseLink as the JPA Provider and HSQLDB as the database. However, you are free to use any JPA provider (like Hibernate, OpenJPA and so on) and database of your choice.


Group ID | Artifact ID | Version
---------|-------------|-------
javax.servlet | servlet-api | 2.5
org.apache.cxf | cxf-rt-frontend-jaxrs | 2.7.5
org.slf4j | slf4j-log4j12 | 1.7.1
junit | junit | 3.8.1
org.apache.olingo | olingo.odata2.api | 1.0.0
org.apache.olingo | olingo.odata2.jpa.processor.api | 1.0.0
org.apache.olingo | olingo.odata2.jpa.processor.core | 1.0.0
org.apache.olingo | olingo.odata2.jpa.processor.ref | 1.0.0
org.apache.olingo | olingo.odata2.core | 1.0.0
org.eclipse.persistence | eclipselink | 2.3.1
org.eclipse.persistence | javax.persistence | 2.0.5
org.hsqldb | hsqldb | 2.2.8
 


Here is a [Sample JPA Model][1] 


##### Create a Dynamic Web Application Project from Scratch:

1. In the command prompt, enter the maven command given here (change the DgroupId and DartifactId as per your requirement)

		mvn archetype:generate -DgroupId=com.sample.jpa -DartifactId=salesorderprocessing.app -DarchetypeArtifactId=maven-archetype-webapp
		
   Maven generates the file system structure for a web application project including a basic POM.xml. This step is completed by creating a Java source folder.
	
2. Create a folder by name 'java' in the path 'src/main/'.

##### Tailor POM.xml 

POM.xml should be modified for adding dependencies like OData Library (Java) and OData JPA Processor Library. Add a dependency to the project that contains JPA models. Open POM.xml and replace the existing content with the following:

		  <?xml version="1.0" ?> 
		  <project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
		    <modelVersion>4.0.0</modelVersion> 
		    <parent>
		      <groupId>org.apache.olingo</groupId> 
		      <artifactId>olingo-odata2-parent</artifactId> 
		      <version>1.0.0</version> 
		      <relativePath>..</relativePath> 
		     </parent>
		     <artifactId>olingo.odata2.jpa.processor.ref.web</artifactId> 
		     <packaging>war</packaging> 
		     <name>${project.groupId}-${project.artifactId}</name> 
		    <dependencies>
             <dependency>
               <!-- required because of auto detection of web facet 2.5 -->
               <groupId>javax.servlet</groupId>
               <artifactId>servlet-api</artifactId>
               <version>2.5</version>
               <scope>provided</scope>
             </dependency>
             <dependency>
               <groupId>org.apache.cxf</groupId>
               <artifactId>cxf-rt-frontend-jaxrs</artifactId>
               <version>2.7.5</version>
             </dependency>
             <dependency>
               <groupId>org.apache.olingo</groupId>
               <artifactId>olingo-odata2-core</artifactId>
               <version>${project.version}</version>
             </dependency>
             <dependency>
               <groupId>org.apache.olingo</groupId>
               <artifactId>olingo-odata2-api</artifactId>
               <version>${project.version}</version>
             </dependency>
             <dependency>
               <groupId>org.apache.olingo</groupId>
               <artifactId>olingo-odata2-jpa-processor-api</artifactId>
               <version>${project.version}</version>
             </dependency>
             <dependency>
               <groupId>org.apache.olingo</groupId>
               <artifactId>olingo-odata2-jpa-processor-core</artifactId>
               <version>${project.version}</version>
             </dependency>
             <dependency>
               <groupId>org.apache.olingo</groupId>
               <artifactId>olingo-odata2-jpa-processor-ref</artifactId>
               <version>${project.version}</version>
             </dependency>
             <dependency>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-log4j12</artifactId>
               <version>1.7.1</version>
             </dependency>
             <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>3.8.1</version>
               <scope>test</scope>
             </dependency>
             <dependency>
               <groupId>org.eclipse.persistence</groupId>
               <artifactId>eclipselink</artifactId>
               <version>2.3.1</version>
             </dependency>
             <dependency>
               <groupId>org.eclipse.persistence</groupId>
               <artifactId>javax.persistence</artifactId>
               <version>2.0.5</version>
             </dependency>
             <dependency>
               <groupId>org.hsqldb</groupId>
               <artifactId>hsqldb</artifactId>
               <version>2.2.8</version>
             </dependency>
           </dependencies>
           <build>
             <finalName>olingo.odata2.jpa.processor.ref.web</finalName> 
           </build>
         </project>

##### Implement an OData Service

The project is now ready to expose OData services. Service Factory provides a means for initializing Entity Data Model (EDM) Provider and OData JPA Processors. Following are the steps for implementing a Service Factory:

1. Add a new Java class by extending ODataJPAServiceFactory.
2. Declare persistence unit name as class variable. For example, private static final String PUNIT_NAME = "persistenceUnitName";
   *Note*: The PUNIT_NAME  refers to the persistence unit name maintained in the persistence.xml of JPA project.
3. Implement the abstract method `initializeODataJPAContext`. Here is the code snippet:

		ODataJPAContext oDataJPAContext = getODataJPAContext();
		oDataJPAContext.setEntityManagerFactory(JPAEntityManagerFactory.getEntityManagerFactory(PUNIT_NAME));
		oDataJPAContext.setPersistenceUnitName(PUNIT_NAME);

##### Configure the Web Application

1. Configure the web application as shown below by adding the following servlet configuration to web.xml. The Service factory which was implemented is configured in the web.xml of the ODataApplication as one of the init parameters.
2. Replace in the following XML with the class name you created in the previous step:

		- <servlet>
		    <servlet-name>JPARefScenarioServlet</servlet-name> 
		    <servlet-class>org.apache.cxf.jaxrs.servlet.CXFNonSpringJaxrsServlet</servlet-class> 
          - <init-param>
               <param-name>javax.ws.rs.Application</param-name> 
               <param-value>org.apache.olingo.odata2.core.rest.app.ODataApplication</param-value> 
            </init-param>
          - <init-param>
               <param-name>org.apache.olingo.odata2.service.factory</param-name> 
               <param-value>org.apache.olingo.odata2.jpa.processor.ref.web.JPAReferenceServiceFactory</param-value> 
            </init-param>
            <load-on-startup>1</load-on-startup> 
          </servlet>
        - <servlet-mapping>
            <servlet-name>JPARefScenarioServlet</servlet-name> 
            <url-pattern>/SalesOrderProcessing.svc/*</url-pattern> 
          </servlet-mapping>
		  
After the implementation, test the web application using http://localhost:8080/olingo.odata2.jpa.processor.ref.web/SalesOrderProcessing.svc
	


  [1]: ../../../resources/Sample_JPA_Model.xml