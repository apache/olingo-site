Title: Implementation of Filter Visitor (JDBC)

# Implementation of Filter Visitor (JDBC)


### How To Guide for implementing a filter tree transformation into a JDBC where clause
The query option $filter can be used to apply a filter query to the result set. This tutorial will be about consuming and working with the filter tree which an application will get from the OData Java library by implementing a transformation of the filter expression into a JDBC where clause. The example explained here will be kept simple to show the mechanism of the visitor pattern. Security problem which occur when using user input (e.g. the filter string of the URI) inside a where clause will be pointed out but not solved for this tutorial. Knowledge about the visitor pattern is not necessary but helpful. If you want to read further please refer to the further information chapter at the end of this tutorial.

### Examples
##### Simple example
If a filter expression is parsed by the OData library it will be transformed into a filter tree. A simple tree for the expression ‘a’ eq ‘b’ would look like this:

![Picture:Simple Filter Expression](/img/FilterExpressionSimple.png)

To visit a filter tree we have to implement the interface `org.apache.olingo.odata2.api.uri.expression.ExpressionVisitor`. For this simple example we will only need the following methods:

- `visitFilter(…)`
- `visitBinary(…)` 
- `visitLiteral(…)` 

These methods will be called if the `accept(…)` method of the filter expression is called. The visitor will always start with the far left of the tree. First visitLiteral is called for 'a' and 'b'  before the `visitBinary()` and finally the `visitFilter()` is called.

Now lets have a look at the implementation of the `visitLiteral()`:

    @Override
    public Object visitLiteral(LiteralExpression literal, EdmLiteral edmLiteral) {
      if(EdmSimpleTypeKind.String.getEdmSimpleTypeInstance().equals(edmLiteral.getType())) {
        // we have to be carefull with strings due to sql injection
        // TODO: Prevent sql injection via escaping
        return "'" + edmLiteral.getLiteral() + "'";
      } else {
        return "'" + edmLiteral.getLiteral() + "'";
      }
    }

The signature for this method contains the literal expression as an object as well as the edmLiteral representation. In this case the literal would be "a" without the '. Inside this method we just return the literal. Since this literal is coming from the URL it represents user input. In the case of the type Edm.String we would have to escape the String to make sure no SQL Injection is possible.

Next the method visitBinary is called:


    @Override
      public Object visitBinary(BinaryExpression binaryExpression, BinaryOperator operator, Object leftSide, Object rightSide) {    
        //Transform the OData filter operator into an equivalent sql operator
        String sqlOperator = "";
        switch (operator) {
        case EQ:
          sqlOperator = "=";
          break;
        case NE:
          sqlOperator = "<>";
          break;
        case OR:
          sqlOperator = "OR";
          break;
        case AND:
          sqlOperator = "AND";
          break;
        case GE:
          sqlOperator = ">=";
          break;
        case GT:
          sqlOperator = ">";
          break;
        case LE:
          sqlOperator = "<=";
          break;
        case LT:
          sqlOperator = "<";
          break;
        default:
          //Other operators are not supported for SQL Statements
          throw new UnsupportetOperatorException("Unsupported operator: " + operator.toUriLiteral());
        }  
        //return the binary statement
        return leftSide + " " + sqlOperator + " " + rightSide;
      }

The signature for this method contains the binaryExpression as an object as well as the left and right Side of this binary expression. In this example the left side would be "a" and the right side would be "b" which we got from the visitLiteral() method. In between we set the operator in its SQL syntax.

Finally the `visitFilter()` method is called:

      @Override
      public Object visitFilterExpression(FilterExpression filterExpression, String expressionString, Object expression) {
          return "WHERE " + expression;
      }

Here we just append the WHERE at the beginning and give the whole thing back to the caller of the `accept()` method.

A simple test can show how the expression is transformed. As one can see the `UriParser` can parse a filter expression if necessary. Usually the application will already have the filter expression.


    @Test
    public void printExpression() throws Exception {
        FilterExpression expression = UriParser.parseFilter(null, null, "'a' eq 'b'");
        String whereClause = (String) expression.accept(new JdbcStringVisitor());
        System.out.println("Raw: " + rawExpression + " ------> Whereclause: " + whereClause);
        System.out.println();
      }

The output will be:


    Raw: 'a' eq 'b' ------> Whereclause: 'a' = 'b'

The implementation right now can only transform literals which will not be sufficient if you want to address a property. If an expression contains properties like "EmployeeId" we have to implement the method `visitProperty()`.

    @Override
    public Object visitProperty(PropertyExpression propertyExpression, String uriLiteral, EdmTyped edmProperty) {
      if (edmProperty == null) {
        //If a property is not found it wont be represented in the database thus we have to throw an exception
        throw new PropertyNotFoundException("Could not find Property: " + uriLiteral);
      } else {
        //It is also possible to use the mapping of the edmProperty if the name differs from the databasename
        try {
          return edmProperty.getName();
        } catch (EdmException e) {
          throw new PropertyNotFoundException(e);
        }
      }
    }

This method has an `edmProperty` in its signature which is set if the `uriLiteral` matches a property in the `EntityType`. Now we could transform an expression like `"EmployeeId eq '1'" or "EmployeeId eq '1' and ManagerId eq '2'"`. To get a validation if the property exists we have to give an `EntityType` on which the filter will be applied. In this case we use the `EntityType` "Employee".

    @Test
    public void printExpressionWithProperty() throws Exception {
      //Use a mocked edmProvider for this tutorial
      TestEdmProvider provider = new TestEdmProvider();
      Edm edm = RuntimeDelegate.createEdm(provider);
      EdmEntityType entityType = edm.getEntityType(TestEdmProvider.NAMESPACE_1, TestEdmProvider.ENTITY_TYPE_1_1.getName());

      String rawExpression = "EmployeeId eq '1'";
      FilterExpression expression = UriParser.parseFilter (null, entityType, rawExpression);
      String whereClause = (String) expression.accept(new JdbcSimpleStringVisitor());
      System.out.println("Raw: " + rawExpression + " ------> Whereclause: " + whereClause);
      System.out.println();
    }

The output will be:


    Raw: EmployeeId eq '1' ------> Whereclause: WHERE EmployeeId = '1'

Test in the sources: JdbcSimpleStringVisitorTest.class

##### Advanced Example
The implementation shown in the previous chapter can transform simple expressions. But if the expression gets more complex like `"'a' eq 'b' or ('c' eq 'd' and 'b' eq 'd')"` it won´t produce a correct where clause. The following test shows this.

    @Test
    public void compareSimpleAndAdvancedVisitor() throws Exception{
      String rawExpression = "'a' eq 'b' or ('c' eq 'd' and 'b' eq 'd')";
      FilterExpression expression = UriParser.parseFilter(null, null, rawExpression);

      String whereClauseSimple = (String) expression.accept(new JdbcSimpleStringVisitor());
      String whereClauseAdvanced = (String) expression.accept(new JdbcAdvancedStringVisitor());
      System.out.println("Simple: " + whereClauseSimple + " ------> Advanced: " + whereClauseAdvanced);
    }

The output will be:


    Simple: WHERE 'a' = 'b' OR 'c' = 'd' AND 'b' = 'd' ------> Advanced: WHERE 'a' = 'b' OR ('c' = 'd' AND 'b' = 'd')

In the simple implementation the brackets will not be transformed. To fix this the method `visitBinary()` has to be enhanced.

    @Override
    public Object visitBinary(BinaryExpression binaryExpression, BinaryOperator operator, Object leftSide, Object rightSide) {
      String actualLeftSide = leftSide.toString();
      String actualRightSide = rightSide.toString();
      if (leftSide instanceof Expression) {
        //If something is lower in the tree and is of the type AND or OR it needs brackets to show the higher priority
        if (BinaryOperator.AND.equals(((Expression) leftSide).getOperator()) || BinaryOperator.OR.equals(((Expression) leftSide).getOperator())) {
        actualLeftSide = "(" + leftSide + ")";
        }
      }
      if (rightSide instanceof Expression) {
        //If something is lower in the tree and is of the type AND or OR it needs brackets to show the higher priority
        if (BinaryOperator.AND.equals(((Expression) rightSide).getOperator()) || BinaryOperator.OR.equals(((Expression) rightSide).getOperator())) {
        actualRightSide = "(" + rightSide + ")";
        }
      }
      //Transform the OData filter operator into an equivalent sql operator
      String sqlOperator = "";
      switch (operator) {
      case EQ:
        sqlOperator = "=";
        break;
      case NE:
        sqlOperator = "<>";
        break;
      case OR:
        sqlOperator = "OR";
        break;
      case AND:
        sqlOperator = "AND";
        break;
      case GE:
        sqlOperator = ">=";
        break;
      case GT:
        sqlOperator = ">";
        break;
      case LE:
        sqlOperator = "<=";
        break;
      case LT:
        sqlOperator = "<";
        break;
      default:
        //Other operators are not supported for SQL  Statements
        throw new UnsupportetOperatorException("Unsupported operator: " + operator.toUriLiteral());
      }

      //return the binary statement
      return new Expression(actualLeftSide + " " + sqlOperator + " " + actualRightSide, operator);
    }

Since simple strings cannot show this complexity a new private class "Expression" was introduced. The signature of all `visit()` methods accept any kind of object so we can mix in the new class. Now we only have to check if one side of the tree is of type String or Expression.

Test in the sources: JdbcAdvancedStringVisitorTest.class

##### Example with prepared Statements
Since string concatenation is very vulnerable against SQL Injection a best practice is to use prepared statements. This can be a tough challenge in this case because not only the value of `EmployeeId` is supplied in the filter expression but the field EmployeeId and the operator as well. Prepared Statements don´t allow statements like `"WHERE ? ? ?"` thus we have to find a way to prepare the prepared statements in advance which can be very complex as the following example will show: The filter expression `"EmployeeId eq '1' and ManagerId eq '2'"` is the same as `"ManagerId eq '2' and EmployeeId eq '1'"` but the prepared statement will always look like `"…. WHERE EmployeeId = ? and ManagerId = ?"`.

This tutorial will not solve this problem. Instead it will show a first idea on how to implement such a prepared statement visitor. Again the Where clause will be created using string concatenation. But this time we will replace literals with a "?". These questionmarks can be set in a prepared statement. The methods `visitLiteral` and `visitProperty` will just return their value while the `visitBinary` will contain the logic.

VisitLiteral:

    @Override
    public Object visitLiteral(final LiteralExpression literal, final EdmLiteral edmLiteral) {
      //Sql Injection is not possible anymore since we are using prepared statements. Thus we can just give back the edmLiteral content
      return edmLiteral.getLiteral();
    }

VisitProperty:

    @Override
    public Object visitProperty(final PropertyExpression propertyExpression, final String uriLiteral, final EdmTyped edmProperty) {
      if (edmProperty == null) {
        //If a property is not found it wont be represented in the database thus we have to throw an exception
        throw new PropertyNotFoundException("Could not find Property: " + uriLiteral);
      } else {
        //To distinguish between literals and properties we give back the whole edmProperty in this case
        return edmProperty;
      }
    }


In addition we will use the following class as a container for the where clause and the parameters:

    public class Expression {
      private String preparedWhere;
      private List<Object> parameters;
      private BinaryOperator operator;

      public Expression(final BinaryOperator operator) {
        preparedWhere = "";
        parameters = new ArrayList<Object>();
        this.operator = operator;
      }

      public void addParameter(final Object parameter) {
      parameters.add(parameter);
      }

      public void setPrepeparedWhere(final String where) {
        preparedWhere = where;
      }

      public List<Object> getParameters() {
        return parameters;
      }

      public BinaryOperator getOperator() {
        return operator;
      }

      @Override
      public String toString() {
        return preparedWhere;
      }
    }

Finally the visitBinary method:

    @Override
    public Object visitBinary(final BinaryExpression binaryExpression, final BinaryOperator operator, final Object leftSide, final Object rightSide) {
      //Transform the OData filter operator into an equivalent sql operator
      String sqlOperator = "";
      switch (operator) {
      case EQ:
        sqlOperator = "=";
        break;
      case NE:
        sqlOperator = "<>";
        break;
      case OR:
        sqlOperator = "OR";
        break;
      case AND:
        sqlOperator = "AND";
        break;
      case GE:
        sqlOperator = ">=";
        break;
      case GT:
        sqlOperator = ">";
        break;
      case LE:
        sqlOperator = "<=";
        break;
      case LT:
        sqlOperator = "<";
        break;
      default:
        //Other operators are not supported for SQL Statements
        throw new UnsupportetOperatorException("Unsupported operator: " + operator.toUriLiteral());
      }
      //The idea is to check if the left side is of type property. If this is the case we append the property name and the operator to the where clause
      if (leftSide instanceof EdmTyped && rightSide instanceof String) {
        Expression expression = new Expression(operator);
        try {
          expression.setPrepeparedWhere(((EdmTyped)   leftSide).getName() + " " + sqlOperator + " ?");
        } catch (EdmException e) {
          throw new RuntimeException("EdmException occured");
        }
        expression.addParameter(rightSide);
        return expression;
      } else if (leftSide instanceof Expression && rightSide instanceof Expression) {
        Expression returnExpression = new Expression(operator);
        Expression leftSideExpression = (Expression) leftSide;
        if (BinaryOperator.AND.equals    (leftSideExpression.getOperator()) || BinaryOperator.OR.equals(leftSideExpression.getOperator())) {
        leftSideExpression.setPrepeparedWhere("(" + leftSideExpression.toString() + ")");
        }
        Expression rightSideExpression = (Expression) rightSide;
        if (BinaryOperator.AND.equals(rightSideExpression.getOperator()) || BinaryOperator.OR.equals(rightSideExpression.getOperator())) {
        rightSideExpression.setPrepeparedWhere("(" + rightSideExpression.toString() + ")");
        }
        returnExpression.setPrepeparedWhere(leftSideExpression.toString() + " " + sqlOperator + " " + rightSideExpression.toString());

        for (Object parameter : leftSideExpression.getParameters()) {
        returnExpression.addParameter(parameter);
        }
        for (Object parameter : rightSideExpression.getParameters()) {
          returnExpression.addParameter(parameter);
        }
        return returnExpression;
      } else {
        throw new RuntimeException("Not right format");
      }
    }


This implementation will work if the filter supplies a property on the left side and a literal on the right side. The output of this new implementation could look like this:

    Raw: EmployeeId eq '1' or (ManagerId eq '2' and TeamId eq '3') ------> Whereclause: EmployeeId = ? OR (ManagerId = ? AND TeamId = ?)
    1
    2
    3


Test in the sources: JdbcPreparedStatementVisitorTest.class

### Further Information
Documentation about how to create such a filter expression can be found under [http://www.odata.org](http://www.odata.org "External Link") in the OData protocol specification.

Visitor pattern: [http://en.wikipedia.org/wiki/Visitor_pattern](http://en.wikipedia.org/wiki/Visitor_pattern "External Link")