Title: Maven Dependencies

# Maven Dependencies

### Consuming OData 2.0 Library

The following dependencies needs to be added to the pom.xml of the consuming project:

```xml
    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-api</artifactId>
      <version>${olingo.version}</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-core</artifactId>
      <version>${olingo.version}</version>
      <scope>runtime</scope>
    </dependency>
```

### Consuming OData 2.0 JPA Processor Extension

To use the JPA Processor Extension following dependencies are required:

```xml
    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-jpa-processor-api</artifactId>
      <version>${olingo.version}</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-api-annotation</artifactId>
      <version>${olingo.version}</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.olingo</groupId>
      <artifactId>olingo-odata2-jpa-processor-core</artifactId>
      <version>${olingo.version}</version>
      <scope>runtime</scope>
    </dependency>
```

${olingo.version} == the concrete version of the library. See also:

* [Download Page](/doc/odata2/download.html)
* [Maven Central](https://search.maven.org/#search|ga|1|org.apache.olingo)
