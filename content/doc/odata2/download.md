Title: Download OData 2.0 Java Library

# Download OData 2.0 Java Library

Apache Olingo OData2 is a collection of Java libraries for
implementing [OData V2][1] protocol clients or servers.

### Release 2.0.13 (2023-10-22)

[Release notes][3]

The Apache Olingo OData2 2.0.13 release is a patch release.

### Commodity Packages

The **Olingo Library Core** packages contains the *API*, the *implementation*, the *documentation as JavaDoc* and the *Reference Scenario*.
The *API* and the according *implementation* can be used in a *production* environment.
The Reference Scenario is a sample and **shall not be used in a production** environment.
The *Core Library* is developed for production environment usage in business scenarios.

Package | zip | Description
--------|-----|------
Olingo OData2 Library            | [Download](https://www.apache.org/dyn/closer.lua/olingo/odata2/2.0.13/olingo-odata2-dist-lib-2.0.13-lib.zip) ([sha512](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-dist-lib-2.0.13-lib.zip.sha512), [pgp](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-dist-lib-2.0.13-lib.zip.asc)) | All you need to implement an OData V2 client or server.
Olingo OData2 Sources            | [Download](https://www.apache.org/dyn/closer.lua/olingo/odata2/2.0.13/olingo-odata2-parent-2.0.13-source-release.zip) ([sha512](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-parent-2.0.13-source-release.zip.sha512), [pgp](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-parent-2.0.13-source-release.zip.asc)) | Olingo OData2 source code.
Olingo OData2 Docs               | [Download](https://www.apache.org/dyn/closer.lua/olingo/odata2/2.0.13/olingo-odata2-dist-javadoc-2.0.13-javadoc.zip) ([sha512](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-dist-javadoc-2.0.13-javadoc.zip.sha512), [pgp](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-dist-javadoc-2.0.13-javadoc.zip.asc)) | Documentation and JavaDoc.
Olingo OData2 Reference Scenario | [Download](https://www.apache.org/dyn/closer.lua/olingo/odata2/2.0.13/olingo-odata2-dist-ref-2.0.13-ref.zip) ([sha512](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-dist-ref-2.0.13-ref.zip.sha512), [pgp](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-dist-ref-2.0.13-ref.zip.asc)) | Deployable WAR files with reference scenario services using <a href="https://cxf.apache.org">Apache CXF</a>.

##### Extension Packages

The **Olingo Library Extension** packages contains extensions which are provided from various contributors in the context of the Olingo open source project.
The extensions provides convenience for easier consumption or creation of an OData service like the *JPA based processor*, the *Java Annotation based processor* or the *Spring Framework integration*.
However the extensions are *not optimized regarding performance or extensibility*.
Interested parties can use the extensions, if they are sufficient for their scenarios in a production environment.
Feature enhancements or optimizations of the extensions have to be done by the interested parties itself.

Package | zip | Description
--------|-----|------
Olingo OData2 JPA Processor      | [Download](https://www.apache.org/dyn/closer.lua/olingo/odata2/2.0.13/olingo-odata2-dist-jpa-2.0.13-jpa.zip) ([sha512](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-dist-jpa-2.0.13-jpa.zip.sha512), [pgp](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-dist-jpa-2.0.13-jpa.zip.asc)) | All you need to expose your JPA model as OData service.
Olingo OData2 Java Annotation Processor      | [Download](https://www.apache.org/dyn/closer.lua/olingo/odata2/2.0.13/olingo-odata2-dist-janos-2.0.13-janos.zip) ([sha512](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-dist-janos-2.0.13-janos.zip.sha512), [pgp](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-dist-janos-2.0.13-janos.zip.asc)) | Use Java Annotations to create a simple OData service for e.g. test cases (without persistence).
Olingo OData2 Spring Extension Sources      | [Download](https://www.apache.org/dyn/closer.lua/olingo/odata2/2.0.13/olingo-odata2-spring-2.0.13-source-release.zip) ([sha512](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-spring-2.0.13-source-release.zip.sha512), [pgp](https://downloads.apache.org/olingo/odata2/2.0.13/olingo-odata2-spring-2.0.13-source-release.zip.asc)) | Support for use of OData library in Spring context.

### Maven

Apache Olingo OData2 artifacts for latest version at [Maven Central](https://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.apache.olingo%22%20AND%20v%3A%222.0.13%22).
For POM dependencies see [here](/doc/odata2/maven.html).

All other Apache Olingo artifacts are also available at [Maven Central](https://search.maven.org/#search|ga|1|org.apache.olingo).

### Older Releases

For older releases please refer to [Archives][4]
or you can get them [using Maven](/doc/odata2/maven.html).

### Verify Authenticity of Downloads package

While downloading the packages, make yourself familiar
on how to verify their integrity, authenticity and provenience
according to the Apache Software Foundation best practices.
Please make sure you check the following resources:

  - [Artifact verification](/verification.html) details
  - Developers and release managers PGP keys are publicly available here: [KEYS](https://downloads.apache.org/olingo/KEYS).


  [1]: https://odata.org
  [3]: https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12314520&version=12344932
  [4]: https://archive.apache.org/dist/olingo/
