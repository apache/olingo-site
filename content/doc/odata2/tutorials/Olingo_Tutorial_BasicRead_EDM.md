Title: How to use EDMX source as EDM Provider within an OData Service 

# How to use EDMX source as EDM Provider within an OData Service 


## How To Guide for the using an EDM Parser

The EDM Parser is designed to parse the metadata document.
To make the parser accessible from the API, we have to implement the method `readMetadata` from interface `org.apache.olingo.odata2.api.ep.EntityProviderInterface`:

     @Override
     public Edm readMetadata(final InputStream inputStream, final boolean validate) throws EntityProviderException {
       EdmProvider provider = new EdmxProvider().parse(inputStream, validate);
       return new EdmImplProv(provider);
      }

The signature contains the `InputStream` that represents a data stream read from a file and flag validate. If validate is set to true, the structure of the metadata will be checked according to the CSDL after parsing. For example: it will be validated that each `EntityType` defines a key element or derives from a `BaseType` that for its part defines a key.

To start the parsing we have to follow the next steps:

- create an object of the class `EdmxProvider`. This class derives from `EdmProvider` and provides implementations for all of its abstract methods 
- invoke the method `parse(InputStream, boolean)` of this object 

Returned is an EDM object that contains the complete information from parsed metadata.

   
