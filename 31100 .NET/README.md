# .NET Review
- [.NET Review](#net-review)
    - [Lecture 1 Linq, enterprise dev practice](#lecture-1-linq-enterprise-dev-practice)
        - [Namespace naming conventions](#namespace-naming-conventions)
        - [LINQ Language INtegrated Query](#linq-language-integrated-query)
    - [Lecture 2 Testing and Refactoring](#lecture-2-testing-and-refactoring)
        - [test approaches](#test-approaches)
        - [Test Driven Development (TDD)](#test-driven-development-tdd)
        - [Dealing with dependencies](#dealing-with-dependencies)
    - [Lecture 3 ASP.NET](#lecture-3-asp-net)
        - [ASP.NET](#asp-net)
        - [Web Server](#web-server)
        - [ASP](#asp)


## Lecture 1 Linq, enterprise dev practice
### Namespace naming conventions
```cs
CompanyName.TechnologyName[.Feature][.Subnamespace]
```
E.g.

```cs
Westdale.PlanManager.WebFrontEnd
```

### LINQ Language INtegrated Query
- SQL could be executed from C# using strings: `db.ExecuteSQL(@”some sql“);`

Advantages of LINQ:
- Behind the scenes conversion to SQL
- Works with non-SQL data sources as well

Prerequisites for LINQ:
- Lambda expressions
- Implicit typing
- Anonymous types
- Extension functions

Delegates (Named method):
- Allows functions to be used as a variable
- Like function pointers but object oriented, type safe and can be associated with specific instance
```cs
public delegate string Formatter(int value)
public static string SthFormat(int value)
public static void foo(Formatter format){
format(1);
}
foo(SthFormat);
```

Anonymous methods:
- unnamed methods, designed to be used with a delegate type.
- the compiler creates an ordinary method with an auto generated name
```cs
Main() {
  Formatter binary = delegate (int x)
  {
    return Convert.ToString(x, 2);
  };
  foo(binary);
}
```

Lambda Expressions:
```cs
Formatter binary = x => Convert.ToString(x, 2);
foo(binary);
```
- Read as “x goes to …” or “input x returns …”

Alternate syntax (Lambda):
```cs
Func<int, string> toBinary = x => Convert.ToString(x, 2);
Func<int, int, int> add = (x, y) => x + y;
Func<int, int, int> subtract = (int x, int y) => x - y;
Func<int, int, int> multiply = (int x, int y) =>
{
  int result = x*y;
  return result;
}
```

Anonymous types:
```cs
var user = new {Name = “Carol”, Age = 35};
```
- Use var
- No class name after new
- Properties names are used in braces {}, read only
- Compiler automatically generates a class of readonly properties

Implicitly typed variables:
- Use var, automatically inferred from the expression by compiler
- Variable is strongly typed
- Object Initializers:

```cs
var user = new User {FirstName = “Carol”, LastName = “Brady”};
```

Collection Initializers:
```cs
var names = new List<string> { “Mike”, “Carol”};
```

Extension Methods:
```cs
namespace MyExtensionMethod
{
  public static class StringConversions {
    public static double ToDouble(this string s) {
      return Double.Parse(s);
    }
  }
}

using MyExtensionMethod;
…
{
  {
    string myString = “6”;
    Console.WriteLine(myString.ToDouble());
  }
}
```
- A static method declared in a static class (first parameter is modified by “this”)
- Called as an instance method of a different class motivation
- “Add” methods to existing types without creating a new derived type
- Share “interface implementation”
- Allow capabilities to be extended (e.g. LINQ) with a simple using statement

what is the point?
- Extension methods allow interfaces (such as IEnumerable and IQueryable) to have implementation

LINQ:
```cs
var namesOfVoters = from c in db.Users
                    where c.Age >= 18
                    select c.LastName;
foreach (var name in namesOfVoters)
{
  Console.WriteLine(name);
}
```

LINQ can query:
- collections/arrays
- databases
- XML
- your own sources

LINQ method syntax:

```cs
var result = numbers.Where(x => x > 5).Select(x => x * x);
```

- the results are available as an IEnumerable<T> collection

## Lecture 2 Testing and Refactoring
### test approaches
black box testing
- when we test the inputs and outputs, but do not look inside
- we check that the system follows its specifications

White box testing
- when we look inside the system as part of the testing
- use our knowledge of the system's internals to test the behaviour
we use **Unit tests, Automated, White box**

### Test Driven Development (TDD)
1. write an automated test that fails
2. write just enough code to get it passing
3. rafactor your code to imporve maintainability

how does this help?
- prevents over-engineering and adding untested behaviours

.NET testing with MSTest:

```cs
[TestClass]
public class UsersTests
{
  [TestMethod]
  public void FullName_Male_StartsWithMr(){
    User user = new User ("Mike", Gender.Male);
    Assert.IsTrue(user.FullName().StartsWith("Mr"), "error msg");
  }
}
```

Test naming convention
- tests for MyProject -> MyProjectTests
- tests for MyClass -> MyClassTests
- individual tests are named [Unit]\_[State]\_[Result]

E.g.
```cs
CalculatorTests::
DivisionTests::
Dividing_FourByZero_Errors
```

Alternate convention
- tests for MyProject -> MyProject.UnitTests
- tests for a unit of work with a spcific state in StateUnderTest
- tests for individual behaviours are simply the behaviour name

E.g.

```cs
Calculator.UnitTests::
DivisionOfTwoNumbers::
Should_throw_an_exception_when_dividing_by_zero
```

Test setup:
`[TestInitialize]` will run before **every** test method

Testing for Exceptions:
`[ExpectedException(typeof(DivideByZeroException))]`
the test will fail if an exception **doesn't** occur

### Dealing with dependencies
your code will often have dependencies on other classes
- file system classes
- database access classes
- web service classes

one appraoch
- class under test uses interface to dependency
- write a dummy interface implementation for testing only
- test code 'injects' fake class
- tests run using the fake class

## Lecture 3 ASP.NET
### ASP.NET
Active Server Pages (ASP).NET
Three Models
- WebForms
  - GUI oriented, using 'forms' and 'controls'
  - a stateful application abstraction over web development
- MVC (Model View Controller)
  - doesn't hide the stateless nature of the web
- Web API
  - supports REST/HTTP APIs
  - very similar approach to MVS

### Web Server
browser <-> server
- HTTP protocol
- TCP port 80 or 443 for HTTPS

Serving static content:

**browser <-> server <-> HTML files**

ASP.NET:

![asp.net](img/lec-2-asp.net.png)

### ASP
Classic ASP (inline) style:
```jsp
<%@ Page Language="C#" %>
<!DOCTYPE html>
<html>
  <body>
    <%
      if (Test())
      {
    %>
      Hello, <%= Expression() %>
    <%
      }
    %>
  </body>
</html>
```