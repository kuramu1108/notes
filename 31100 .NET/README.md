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
        - [ASP.NET WebForms:](#asp-net-webforms)
        - [Web.config](#web-config)
        - [Events](#events)
        - [State](#state)
        - [Redirect](#redirect)
    - [Lecture 4 More ASP.NET WebForms and Architecture](#lecture-4-more-asp-net-webforms-and-architecture)
        - [Page life-cycle](#page-life-cycle)
        - [Life-cycle events](#life-cycle-events)
        - [Request and Response](#request-and-response)
            - [a note on testing](#a-note-on-testing)
        - [Redirect vs Transfer](#redirect-vs-transfer)
        - [Sessions and Cookies](#sessions-and-cookies)
        - [Authentication/Authorization](#authentication-authorization)
        - [Master pages](#master-pages)
        - [User controls](#user-controls)
        - [Application architectures](#application-architectures)
            - [1-tier](#1-tier)
            - [2-tier](#2-tier)
            - [3-tier](#3-tier)
            - [4-tier](#4-tier)
    - [Lecture 5 Data Access](#lecture-5-data-access)
        - [ADO.NET](#ado-net)
        - [ADO.NET Layers](#ado-net-layers)
            - [Connected Layer](#connected-layer)
            - [Disconnected layer](#disconnected-layer)
                - [Typed DataSets](#typed-datasets)
        - [Transactions](#transactions)
        - [DataReader vs DataSet](#datareader-vs-dataset)
    - [Lecture 7](#lecture-7)


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
```html
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

### ASP.NET WebForms:
ASPX:
```html
<%@ Page Language="C#"
    AutoEventWireup="true"
    CodeBehind="User.aspx.cs"
    Inherits="WebApplication.User"  %>
    <!DOCTYPE html>
<html>
<body>
<form runat="server">
        <asp:TextBox ID="FirstName" runat="server">
            </asp:TextBox>
        <asp:Button ID="Button" runat="server"
    </form> Text="Save" OnClick="Button_Click" />
</body>
</html>
```

- server-side controls have a `runat="server"` attribute
- standard ASP.NET controls are prefixed with `asp:`
- server-side event handlers can be defined in the ASPX

Code behind:
```cs
using System;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using System.Diagnostics;
namespace WebApplication
{
  public partial class User : System.Web.UI.Page
  {
    protected void Page_Load(object sender, EventArgs e)
    {      
      // initialization
    }
    protected void Button_Click(object sender, EventArgs e)
    {
      Debug.WriteLine(FirstName.Text);
    }
  } 
}
```

- a partial class
- `Page_Load` is called before each page request
- `Button_Click` is called whenever the button is clicked

Designer Class:
- another partial class auto generated by Visual Studio
- it declares the user control

### Web.config
```xml
<appSettings>
  <add key="ValidationSettings:UnobtrusiveValidationMode" value="None"/>
  <add key="MinimumBalance" value="150"/>
</appSettings>
```
```cs
using System.Web.Configuration;
...
string minimumBalanceString = WebConfigurationManager.AppSettings["MinimumBalance"];
int minimumBalance = int.Parse(minimumBalanceString);
```

why is this useful?
- WebConfigurationManager is typically used in web applications where it may deal with hierarchies of web.config files
- the configuration files are on a per-project basis
- a separate App.config file would be needed if used in unit test project

### Events
Multicast events
```cs
Button.Click += Button_Click;

// behind the scene declarations
// in System namespace
public delegate void EventHandler(object sender, EventArgs e);
// in the System.Web.UI.WebControls.Button class
public event EventHandler Click;
```

### State
- Session
  - SessionState is persisted on the server for each browser session
- ViewState
  - ViewState is sent between the server and client on every request

store value in the Session and ViewState object:
```cs
Session["somekey"] = 123;
int value = （int) Session["somekey"];
```

### Redirect
```cs
Response.Redirect("Other.aspx");
Server.Transfer("Other.aspx");
```

`Response.Redirect` is more preferred as `Server.Transfer` does the redirect on the server so may result in misleading URLs
e.g. A request to PageA.aspx, if transferred to PageB.aspx will retain the URL of PageA.aspx.

## Lecture 4 More ASP.NET WebForms and Architecture
### Page life-cycle
High Level understanding:
- requested received
- object instantiated
- page set-up
- `Page_Load` is called
- control events are fired
- response is sent to user

Full page life-cucle:
- Page request
- Start
  - PreInit
- Initialization
  - Init
  - InitComplete
  - PreLoad
- Load
  - Load
- Postback events
  - Control events
  - LoadComplete
  - PreRender
  - PreRenderCoimplete
  - SaveStateComplete
- Rendering
  - Render
- Unload
  - Unload

### Life-cycle events
Three approaches
- AutoEventWireup
```cs
protected void Page_Load(object sender, EventArgs e)
```
if .aspx file having `AutoEventWireup="True"` define in the `<%@Page %>` tag, can write custom event handlers, just by defining a function prefixed by `Page_`
- Delegates
```cs
Load += Listner;
```
programmatically wire up a delegate using event handlers, but may need to do this early (e.g. in constructor)
- Virtual methods
```cs
protected override void OnLoad(EventArgs e)
{
  base.OnInit(e);
}
```
directly override the event methods in `System.Web.UI.Page`, however, it is important to call the base event, otherwise the page may not be rendered properly

### Request and Response
- instance of each is available in a Page
- Request
  - HTTP GET/POST request properties
  - Query string, cookies, parameters
- Response
  - Send HTTP response properties
  - Redirects, cookies, HTML, headers

```cs
// Request
Request.Headers["User-Agent"];
Request.HttpMethod;
Request.Params["Name"];

//Response
Response.Write("<h3>Hello world!</h3>");
Respose.ContentType = "text/plain";
Response.Headers.Add("Debug-Info", "v1.0");
Response.Redirect("Welcome.aspx");
```

#### a note on testing
- Request and Response both interact with HttpContext
- all are static objects
- it is very hard to stub/mock a static object
- tips for unit testing with WebFormsL
  - separate business logic from the page
  - encapuslate interactions with Request and Response
  - consider using an MVP(Model-View-Presenter) pattern (advanced)

### Redirect vs Transfer
`Response.Redirect("Other.aspx")`
- causes HTTP Code 302 to be sent
- browser then requests the page
- all form values, etc from current page are lost

`Server.Transfer("SomePage.aspx")`
- direct transfer (no new request from browser)
- form values can be preserved if required
- URL in browser is unchanged

### Sessions and Cookies
Cookies
- a text file stored on the **client's** computer by the web browser
- the web site uses the cookie to store information about the user
- sent with each request from the user to the web site
- sent as a HTTP header

```cs
HttpCookie cookie = new HttpCookie("user", "Cindy");
cookie.Expires = DateTime.Now.AddMinutes(10.0);
Response.SetCookie(cookie);

Request.Cookies["user"].Value;
```

Limitations:
- can only store strings
- can be disabled by the user
- at least (at most) 300 cookies total
- at least (at most) 4096 bytes per cookie
- at least (at most) 20 cookies per domain

HttpSessionState
- available in every page via `Session`
- stores objects by key (name-value pairs)
- Users cookies behind-the-scenes `ASP.NET_SessionId=...;`
- Fall-back to URL encoding

ViewState
- `ViewState` is similar to `Session`
- implemeted as a hidden form variable (doesn't depend on cookies)
- available on post back (not sessions)
- is transferred with every requet and respose

### Authentication/Authorization
Authentication strategies
- no authentication
- individual user accounts
for general public
- organizational accounts
use with Active Directory
- windows authentication
for small Local Area Network

Authentication technologies
- membership (old)
- identity (new)
- cookie authentication

when first login, login credentials are checked and authentication information is added to browser cookie, so no need to sent credential with every request

Authentication concepts
- Users - individual
- Roles - category
- Claims - store user information
- Authentication - who are u
- Authorization - what r u allowed to do

Getting the current user:
used the `User` Proerty of `System.Web.UI.Page`
```cs
User.Identity.Name
User.IsInRole("Admin")
```

Protecting files/directories:
in web.config
```xml
<location path="Admin.aspx">
  <system.web>
    <authorization>
      <allow roles="Admin"/> 
      <deny users="*" />
    </authorization>
  </system.web>
</location> 
```

- the asterisk (*) means "everyone"
- the question mark(?) means "unauthenticated users"

### Master pages
- use master pages to achieve a common look and feel
- master pages use placeholder tags for positioning nested content
- content pages "inherid" common GUI elements from master pages
- known as "visual inheritance"

Master:
```html
<%@Master Language="C#" %>

<asp:ContentPlaceHolder ID="Main" runat="server"/>
```

Content Pages:
```html
<%@Page Language="C#" MasterPageFile="Site.Master" %>

<asp:Content ContentPlaceHolderID="Main" runat="server">
  Welcome!
</asp:Content>
```

### User controls
web applications often reuse functionality on multiple pages:
- user controls (ASCX) can achieve this
- custom components can be embedded like ordinary components

```html
<%@Control Language="C#" CodeBehind="MyControl.ascx.cs" Inherits="App.MyControl" %>

<asp:TextBox ID="FirstName" runat="server"></asp:TextBox>
<asp:TextBox ID="LastName" runat="server"></asp:TextBox>
```
Content Pages:
```html
<%@Page Language="C#" %>

<%@ Register Src="~/MyControl.ascx" TagPrefix="uc" TagName="MyControl" %> 
<uc:MyControl ID="UserDetails" runat="server"/>
```
### Application architectures
####1-tier

![1-tier](img/lec-4-1-tier.png)

Advantages
- Extremely simple

Diadvantages
- difficulty in re-using code if presentation method needs to be changed
- testing is more complex
- difficult to share functionality
- functionally unrelated code is contained in a single class
- no shared data

####2-tier

![2-tier](img/lec-4-2-tier.png)

Advantages
- Conceptually simple
- Shared data

Disadvantages
- diffculty in re-using code if presentation method needs to be changed
- testing is more complex
- difficult to share functionality
- functionally unrelated code is contained in a single class

####3-tier

![3-tier](img/lec-4-3-tier.png)

Advantages
- easier testing
- easier to modify user interface
- more scalable
- data access classes can be shared with other applications

Disadvantages
- More complex

####4-tier

![4-tier](img/lec-4-4-tier.png)

Advantages
- best for testing
- more scalable
- interface classes are only responsible for displaying data (easy to change interface)
- one place to change business logic
- business and data access functions can be chared easily

Disadvantages
- more complex
- less support from IDE tools

## Lecture 5 Data Access
### ADO.NET
- a set of base classes/interfaces and optimized implementations of those interfaces
- e.g. to access an SQL server database, you would use the ADO.NET data provider that is optimized for SQL server

```cs
var settings = System.Configuration.ConfigurationManager.ConnectionStrings["Db"]; 
DbProviderFactory factory = DbProviderFactories.GetFactory(settings.ProviderName); 
DbConnection conn = factory.CreateConnection(); 
conn.ConnectionString = settings.ConnectionString;
conn.Open();
```

### ADO.NET Layers
- connected
  - the connected layer maintains a datavse connection while working with the datavse
  - the traditional approach to interfacing with database
  - closely matches the low-level interfaces that most database provide
- disconnected
  - the disconnected layer creates an in-memory copy of part of the datavase
  - designed to conserve connections, increase performance and also reduec programmer errors in managing conections
- object-relational (EntityFramework)
  - in-memory view of the database
  - however, it is not relational - it does not have tables and columns
  - instead it automatically translates tables into ordinary objects

#### Connected Layer
- DbConnection (SqlConnection)
- DbCommand (SqlCommand)
- DbataReader (SqlDataReader)

Querying the database
- create a connection
- open the connection
- create a command
- execute the command
- read the result
- close the connection

Create a connection:
- select a data provider (e.g. `System.Data.SqlClient`)
- create a connection instance
- pass a connection string to configure the connection

Connection strings:
-  are preferably stored in .config files

```xml
<connectionStrings>
<add name="Db"
  connectionString="Data Source=(LocalDB)\v11.0;Integrated Security"
  providerName="System.Data.SqlClient"/>
</connectionStrings>
```
```cs
using System.Configuration; 
ConfigurationManager.ConnectionStrings["Db"].ProviderName 
ConfigurationManager.ConnectionStrings["Db"].ConnectionString
```

Commands:
```cs
SqlConnection c = new SqlConnection();
c.ConnectionString = connectionString;
c.Open();
string query = "select count(*) from Customers";
SqlCommand command = new SqlCommand(query);
command.Connection = c;
Response.Write(command.ExecuteScalar().ToString()); 
c.Close();
```

Using `using`:
```cs
using (SqlConnection c = new SqlConnection())
{
  c.Open();
  ....
  // c.Close(); no need
}
```

- the `using` statement automatically ensures that connections are closed
- it could be used with any class that implements `IDisposable`

Command:

four methods to execute commands:
- `ExecuteScalar` - when the query will return single value
- `ExecuteReader` - Select statements, returns a  `DbDataReader`
- `ExecuteNonQuery` - insert, delete, update statements, not results
- `ExecuteXMLreader` - for handling XML from SQL server

DataReader:
- read only, forward only data stream
- useful when you only need to access data once
- retains a database connection

```cs
using (SqlDataReader result = command.ExecuteReader())
{
  while (result.Read()) 
  {
    Console.WrilteLine(result.GetString(0));
    Console.WrilteLine(result.GetInt32(1));
  }
}
```

Parameters:
```cs
string query = @"select FirstName, LastName
                  from SystemUser
                  where LastName = @name";
Sqlcommand command = new SqlCommand(query, conn)
command.Parameters.Add(new SqlParameter("name", "Robert"));
```

#### Disconnected layer
why?
- memory is cheap and fast
- database connections are sloq and resource consuming
- simple and clean
- ensures that connections are closed quickly
- allow results to be cached easier
- reduce the possibility that connections are not closed

Disconnected Layer:
- `DataSet` - an "in-memory" database
- `DataTable` - a table in the `DataSet`
- `DataRow` - a row of a `DataTable`
- `DataAdapter` - connects a `DataSet` to a database

Querying with DataSets:
- create `DataAdapter` with SQL string and connection
- create a new `DataSet`
- fill dataset using `DbDataAdapter.Fill(dataset`
- process or display DataSet 'offline'

![dataset](img/lec-5-dataset.png)

```cs
DataSet result = new DataSet();
string query = "select FirstName, LastName, DateOfBirth from SystemUser"; 
SqlDataAdapter adapter = new SqlDataAdapter(query, connectionString); 
adapter.Fill(result);
// First table, first row, first column
Console.WriteLine(result.Tables[0].Rows[0][0]);
```

Updating with DataSets:
- make changes to `DataSet`  
- update database using `DbDataAdapter.Update(dataset)`
- Note:
  - you need to specify the update queries (`DbDataAdapter.UpdateCommand`)
  - However, `SqlCommandBuilder` can do this automatically

##### Typed DataSets
- simply a custom subclass of `DataSet`/`DataTable`

```cs
SystemUserTable users = new SystemUserTableAdapter().GetData();
```

### Transactions
Group database updates into a single unit
- 'All or nothing'
- isolation
- e.g. Bank account transger
- user `DbTransaction (SqlTransaction)`

```cs
// BeginTransaction returns an instance of DbTransaction
var tx = connection.BeginTransaction();
tx.Commit();
tx.RollBack();
tx.Close();

// actual usage
using (var tx = conn.BeginTransaction())
{
  var c1 = new SqlCommand(update1, conn, tx);
  c1.ExecuteNonQuery();

  var c2 = new SqlCommand(update2, conn, tx);
  c2.ExecuteNonQuery();

  tx.Commit();
}
// the code will be run 'all or nothing'
// if either command fails, then both will be cancelled
```

### DataReader vs DataSet
- DataSets are very easy to use and reduce errors
  - however it create a tendency to use them pervasively throughout an application
  - this may be a problem if you want your design to be independent of the relational database
- while DataReaders are more difficult to work with, their low level abstraction may encourage better separation and isolation

## Lecture 7