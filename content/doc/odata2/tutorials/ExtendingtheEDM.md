Title: Extending the EDM Generated from the JPA Models

# Extending the EDM Generated from the JPA Models

The Entity Data Model (EDM) generated from the JPA models can be extended with new Entity Types, Complex Types and also the existing EDM elements can be modified by implementing the interface method `org.apache.olingo.odata2.jpa.processor.api.model.JPAEdmExtension.extendJPAEdmSchema`.

Link to the code - [Support EDM Extension][1]

#### How to Add a Complex Type to an EDM

Consider a scenario where we have a Plain Old Java Object (POJO) in the OrderValue Java class and let us try to transform this POJO into a Complex Type in the EDM.

###### POJO in OrderValue Java Class

        public class OrderValue
		{

		private double amount;
		private String currency;

		public double getAmount()

		{ return amount; }
		public void setAmount(double amount)

		{ this.amount = amount; }
		public String getCurrency()

		{ return currency; }
		public void setCurrency(String currency)

		{ this.currency = currency; }

		}


Here is an example where the `SalesOrderProcessingExtension` implements the `JPAEdmExtension`.

##### Sample Code

                @Override
		public void extendJPAEdmSchema(final JPAEdmSchemaView view) {
		  Schema edmSchema = view.getEdmSchema();
		  edmSchema.getComplexTypes().add(getComplexType());
		}

		private ComplexType getComplexType() {
		  ComplexType complexType = new ComplexType();

		  List<Property> properties = new ArrayList<Property>();
		  SimpleProperty property = new SimpleProperty();

		  property.setName("Amount");
		  property.setType(EdmSimpleTypeKind.Double);
		  properties.add(property);

		  property = new SimpleProperty();
		  property.setName("Currency");
		  property.setType(EdmSimpleTypeKind.String);
		  properties.add(property);

		  complexType.setName("OrderValue");
		  complexType.setProperties(properties);

		  return complexType;

		}

You need to set your `JPAEDMExtension` type in the `initializeODataJPAContext` of `JPAReferenceServiceFactory`.

The Complex Type added to the EDM can be used as Function Imports Return Type. See [Adding Function Imports to OData Services][2] for more information.





  [1]: https://gitbox.apache.org/repos/asf?p=olingo-odata2.git;a=blob;f=odata2-jpa-processor/jpa-web/src/main/java/org/apache/olingo/odata2/jpa/processor/ref/extension/SalesOrderProcessingExtension.java;h=3dacd7e727528cb79cb3d4a878ac53d0a4b25277;hb=ecdc476
  [2]: </doc/odata2/tutorials/jpafunctionimport.html>
