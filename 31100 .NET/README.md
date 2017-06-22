# .NET Review
## Lect 1 Linq, enterprise dev practice

### Namespace naming conventions
```C#
CompanyName.TechnologyName[.Feature][.Subnamespace]
```
E.g.
```C#
Westdale.PlanManager.WebFrontEnd
```

### LINQ Language INtegrated Query
- SQL could be executed from C# using strings: db.ExecuteSQL(@”some sql“);

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
```C#
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
```C#
Main() {
    Formatter binary = delegate (int x)
    {
        return Convert.ToString(x, 2);
    };
    foo(binary);
}
```

Lambda Expressions:
```C#
Formatter binary = x => Convert.ToString(x, 2);
foo(binary);
```
- Read as “x goes to …” or “input x returns …”

Alternate syntax (Lambda):
```C#
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
```C#
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
```C#
var user = new User {FirstName = “Carol”, LastName = “Brady”};
```

Collection Initializers:
```C#
var names = new List<string> { “Mike”, “Carol”};
```

Extension Methods:
```C#
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
