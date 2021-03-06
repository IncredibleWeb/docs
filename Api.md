# API
This chapter describes the architecture of the API which serves the JSON content.

## Prerequisites
- Microsoft Visual Studio
- MSSQL & Microsoft SQL Management Studio

## Skills needed:
- C#
- .NET
- MVC

## Naming Conventions
PascalCasing for class, property and method names.
```csharp
public class TestController
public void TestCount()
```
PascalCasing for abbreviations 3 characters or more. If the abbreviation is made of 2 chars they are both uppercase.
```csharp
private UIControl uiControl;
private TestApi testApi;
```
camelCasing for method arguments and local variables.
```csharp
public void MyFunction(int testId, string testTitle)
var testObject = new Object();
```
Predefined type names
```csharp
int jobId;
string jobTitle;
bool isCached;
```
`var` for local variable declarations.
```csharp
var myItem = new List<Object>();
```
Noun or noun phrases for class names.
```csharp
public class Department
public class JobController
```
Prefix interfaces with I. Interface names are nouns, noun phrases or adjectives.
```csharp
public interface IPageViewModel
public interface ITestable
```
Organize namespaces with a clearly defined structure
```csharp
using System;
using System.Linq;
using System.Web.Mvc;
using Umbraco.Web;
using Umbraco.Web.Mvc;
using Umbraco.Core.Models;
using Company.ViewModels.Base;
using Company.ViewModels.Page;
using Company.ViewModels.Home;
```
Vertically align curly brackets
```csharp
public class HomePage
{
    public Object TestObj { get; set; }
{
```
Static variables should be declared at the top of the class, member variables afterwards, then the constructor and finally methods.
```csharp
public class Member
{
    public static string FirstName;
    public static string LastName;

    public int PhoneNumber { get; set; }
    public string Address { get; set; }

    public Member()
    {
    }
    
    public MemberAccount()
    {
    }
}
```
All methods in the API should return IHttpActionResult
```csharp
public IHttpActionResult Get()
{
    //Code here
}
```
## Models vs DTOs
Models are the object defenitions we're using to express a number of domains within our solution which will be used by our APIs to receive or send information. Models should as much as possible be domain driven ex Product, Customer, Page etc.

DTOs by definition are objects used to transfer data between layers in a multi-tier architecture. Often DTOs are also used by APIs to expose information. In our case in order to distinguish between our own models and 3rd party models we will be using DTOs to define 3rd party object definitions.

## Model Structure
Models should be grouped by domains in the form of a project folder ex Products, News, Contacts. Moreover Models should be further grouped into Custom and SDK models. The idea is to have common models whicha are frequently required in Umbraco based applications grouped together so that we can easily identify which code is reusable and which isn't.

Some models may need to be further customised to cater for specific requirements pertaining to the project. In which case we will create the custom object in the Custom folder under the same domain as in the SDK folder and prepend 'Custom' to it. The custom object should inherit the SDK object.

```
Models
    - Custom
        - About
        - Careers
        - News
        - Products
            - CustomProductsPage.cs
    - Sdk
        - Contacts
        - News
        - Pages
        - Products
            - Product.cs
            - ProductsPage.cs
```

## REST Conventions
`GET /users` - Retrieves a list of users<br />
`GET /users/36` - Retrieves a user with UID 36<br />
`GET /users/36/orders` - Retrieves a list of orders made by user with UID 36<br />
`GET /users/36/orders/4` - Retrieves order with UID 4 made by user UID 36<br />
`GET /users/?name=john-doe` - Filters through all the users and retrieves only the users with name John Doe.

`POST /users` - Creates a new user<br />
`POST /users/36/orders` - Creates a new order belong to user with UID 36

`PUT /users/36` - Updates user with UID 36 (**Note that `PUT` will replace the entire object**)

`PATCH /users/36` - Partially updates user with UID 36

`DELETE /users/36` - Deletes user with UID 36

## Guidlines for Umbraco APIs
1) Organise the Controller APIs in SDK, Custom and Shared folders
2) Content which is required in multuple controllers should be moved to the BasePageController when possible
3) Services should be used to encapsulate logic which is re-used in APIs
4) Wherever possible, Services should return IQueryable objects so as to allow further query manipulation at Controller level
5) Controller APIs should be kept as lean as possible and Services should contain most of the logic (its easier to maintain Services through DI than API endpoints)
6) Do not use REST conventions for non-REST calls. 
    - PageController: api/page/{url} - returns one page with a unique URL identifier - Correct
    - AboutController: api/about - returns a list with one about page item - Incorrect and should be replaced by api/about/getpage
7) Page Controllers should have a 'getPage' endpoint which will return all the page data required
8) Regular Controllers should respect REST guidlines and expose domain specific CRUD operations apart from custom methods
9) In REST contollers and methods put id as part of url
    - api/products/{id}
10) Do not use the same controller for different purposes ex to return a list but also to return one item, keep them seperate.
11) Use meaningful return status codes ex. StatusCode(HttpStatusCode.BadRequest)

## Umbraco Setup
- Create a new Visual Studio solution and Empty Project, including dependencies for MVC. Use the naming convention, `MyProject.Api` for the project.
- Install [UmbracoCMS](https://www.umbraco.com) using Nuget Package Manager.
`PM> Install-Package UmbracoCms`
- After the packages have completed installation, you are required to build the project.
- Setup a new IIS website pointing to the root directory of your project and assign it a unique and recognizable hostname in the format `my-api.localhost`. Depending on your OS, you may be required to add the URL to your hosts file (OSX: `sudo nano /etc/hosts`; Windows: `C:\Windows\System32\drivers\etc\hosts`).
- Navigating to your new URL, should load the Umbraco setup.
- Complete the necessary fields and then choose `Customize`.
- From the dropdown select Microsoft SQL Server
- Create an empty SQL database on your local MSSQL server
- Configure Umbraco to connect to your new database
- Do not install any starter packs.
- After setup is completed, you should be able to access Umbraco using the path `/umbraco'. 

### UmbracoSettings.config
- Set the node ID for `<error404>` to the home page. This will remove the "This page is intentionally left ugly" error when throwing a 404.
- Set the `<notifications/email>` to `support@incredible-web.com`

### Additional Setup
- Install Microsoft.AspNet.WebApi.Cors
`PM> Install-Package Microsoft.AspNet.WebApi.Cors`
- Setup global.asax
`<%@ Application CodeBehind="Global.asax.cs" Inherits="MyProject.Api.Global" Language="C#" %>`
- Setup global.asax.cs
```csharp
    public class Global : Umbraco.Web.UmbracoApplication
    {
        protected override void OnApplicationStarting(object sender, EventArgs e)
        {
            base.OnApplicationStarting(sender, e);
            WebApiConfig.Register(GlobalConfiguration.Configuration);
        }

        protected override void OnApplicationError(object sender, EventArgs e)
        {
            var error = HttpContext.Current.Server.GetLastError();
            LogHelper.Error(sender.GetType(), error.Message, error);
        }
    }
```
- Setup the App_Start/WebApiConfig.cs
```csharp
public static void Register(HttpConfiguration config)
{
    // json formatting rules
    config.Formatters.JsonFormatter.SerializerSettings = new JsonSerializerSettings
    {
        ContractResolver = new CamelCasePropertyNamesContractResolver(),
        NullValueHandling = NullValueHandling.Ignore
    };

    // Web Api routes
    config.MapHttpAttributeRoutes();

    // enable CORS
    var cors = new EnableCorsAttribute("localhost:3000", "*", "*");
    config.EnableCors(cors);
    
    // disable XML formatter
    config.Formatters.Remove(config.Formatters.XmlFormatter);
}
```
- Install UmbracoFileSystemProviders.Azure to store files on Azure rather than locally on each machine.
`PM> Install-Package UmbracoFileSystemProviders.Azure -IncludePrerelease`
After installation, navigate to the file Config/FileProviders.config and update the storage connection string to the Azure storage account.

- Exclude Umbraco, Umbraco_Client and Views folders from the project

## Configurations
### Web.config
Update the `web.config` keys and settings as below
```csharp
    <appSettings>
        <add key="Umbraco.ModelsBuilder.Enable" value="false"/>
    </appSettings>
    <system.web>
        <customErrors mode="Off" />
    </system.web
    <system.net>
        <mailSettings>
            <smtp>
                <network host="127.0.0.1" port="2525" />
            </smtp>
        </mailSettings>
    </system.net>
```
Add the following `appSettings` as per below:
```csharp
    <appSettings>
        <add key="vs:EnableBrowserLink" value="false" />
        <add key="Defaults:Email" value="info@my-api.com" />
        <add key="Defaults:Sender" value="My API" />
        <add key="Defaults:TestEmail" value="support@incredible-web.com" />
        <add key="Defaults:IsTestMode" value="false" />
        <add key="Defaults:IsSMTP" value="true" />
        <add key="Site:Title" value="My API" />
        <add key="Site:Url" value="http://my-api.localhost" />
    </appSetting>
```
Add the following inside `httpProtocol/customHeaders`
```csharp
    <remove name="X-Frame-Options" />
    <add name="X-Frame-Options" value="sameorigin" />
```
### Web.release.config
Replace the entire `web.release.config` with the following code which will only be included when the application is published to production.
```csharp
<?xml version="1.0" encoding="utf-8"?>

<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">

  <system.web>
    <compilation xdt:Transform="RemoveAttributes(debug)" />
    <customErrors mode="RemoteOnly" defaultRedirect="500.html" xdt:Transform="Replace">
    </customErrors>
  </system.web>

  <system.webServer>
    <httpCompression xdt:Transform="InsertIfMissing" directory="%SystemDrive%\inetpub\temp\IIS Temporary Compressed Files">
      <scheme name="gzip" dll="%Windir%\system32\inetsrv\gzip.dll"/>
      <dynamicTypes>
        <add mimeType="text/*" enabled="true"/>
        <add mimeType="message/*" enabled="true"/>
        <add mimeType="application/javascript" enabled="true"/>
        <add mimeType="*/*" enabled="false"/>
      </dynamicTypes>
      <staticTypes>
        <add mimeType="text/*" enabled="true"/>
        <add mimeType="message/*" enabled="true"/>
        <add mimeType="application/javascript" enabled="true"/>
        <add mimeType="*/*" enabled="false"/>
      </staticTypes>
    </httpCompression>
    <urlCompression xdt:Transform="InsertIfMissing" doStaticCompression="true" doDynamicCompression="true"/>
    <httpErrors xdt:Transform="InsertIfMissing" existingResponse="PassThrough" />
    <staticContent>
      <clientCache xdt:Transform="InsertIfMissing" cacheControlMode="UseMaxAge" cacheControlMaxAge="7.00:00:00" />
    </staticContent>
    <rewrite xdt:Transform="InsertIfMissing">
      <rules>
        <rule name="Redirect HTTP to HTTPS" stopProcessing="true">
          <match url="(.*)" />
          <conditions>
            <add input="{HTTPS}" pattern="off" />
          </conditions>
          <action type="Redirect" url="https://{HTTP_HOST}/{R:0}" redirectType="Permanent" />
        </rule>
        <rule name="Redirect to WWW" stopProcessing="true">
          <match url=".*" />
          <conditions>
            <add input="{HTTP_HOST}" pattern="^mywebsite\.com$" />
          </conditions>
          <action type="Redirect" url="https://www.mywebsite.com/{R:0}" redirectType="Permanent" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

### Routing
The current setup does not allow the use of default WebApi routes because of conflicts with Umbraco's back office. Therefore all routes are required to be setup using attribute routes.
```csharp
[Route("/api/orders")]
public IHttpActionResult Get(int id = 0)
```
### Logging
Logging is done using Umbraco's `LogHelper` as shown in the global.asax.cs above. Log traces are stored in `App_Data\Logs`. If a method requires additional logging, it should be done similarly by using the `LogHelper` methods.

## Document Types
The document type is the definiton for your content, but it is not the actual data itself. Creating document types is made by opening the Settings tab in the Umbraco Backoffice and right clicking "Document Types" button. The structure at the moment is a combination of nested document types and compositions.

### Nested Document Types
Nested document types have hierarchy in the backoffice and that makes it easier to see the structure. Creating a Child DocType is done by right clicking the Parent DocType.

Example: Page (DocType) -> Home (DocType)

### Compositions
Compositions are modular document types. This means that every document type which is part of the compositions has properties which can be inherited by other document types. It is mainly used for base document types like Page, Meta and more.

Inheriting composition document is done by opening a document type (for example Page) and clicking on the tab Compositions. All the DocTypes that are available will be shown there. When a document type inherits from another one it will get all properties from it and use them as own. 

***NB!*** Document Type properties must be unique in the whole backoffice otherwise there will be conflicts when inheriting compositions.

### Base DocTypes
- [Page](https://github.com/IncredibleWeb/architecture/blob/master/DocTypes/page.udt)


## Responsive Images
The web applications will make use of [srcset](https://www.w3.org/TR/html-srcset/) to display responsive images. The API is responsible for responding with an array of the difference sizes/version of the same image. Each image will observe the following structure:
```
[{
    title: "banner.jpeg",
    imageUrl: "/media/1033/banner.jpeg",
    alternateText: "banner.jpeg",
    width: "640",
    height: "416"
}, {
    title: "banner-2x.jpg",
    imageUrl: "/media/1034/banner-2x.jpg",
    alternateText: "banner-2x.jpg",
    width: "1440",
    height: "936"
}]
```

## Continuous Deployment
Visual Studio Online (VSO) and Visual Studio Team Services serve as a complete eco system for developers to plan, design, store, build, deploy and test their projects.
At Incredible Web we use a GIT version control system to store and version our projects' source code. VSO provides us with private on the cloud remote GIT repositories where we currently store all our GIT repositories.
VSO also provides a build and continous integration environment which will allow us to automatically build and release source code available on these private GIT cloud repositories and release them to our Azure web app slots.
The process takes place into 2 steps:
- Build Process
- Release Process

### Build Process
In this process we decide what to build, when, how and the final artifact resulting from the build.
**What**: Specify the repository which the Build process will build
**When**: Specify what triggers a build process
**How**: Specify the pipeline of processes which should take place when building

### Pipeline of Processes
Given our current development stack, whenever a build is triggered we execute the following processes:
- Get Sources
- Nuget Restore
- Build Solition (VS Build)
- Publish Symbols Path
- Publish Artifact

### Release Process
The release process will grap the published artifact and release it to a deployment environment such as an Azure web app.
A release defenition expects an artifact from a Build definition and deploys it to an environment such as a web app.

## Drawbacks
The major drawbacks from the proposed architecture is that Umbraco libraries are no longer available on the rendering of content, therefore all responses from the API should already be rendered for to be displayed by the server or client side applications.

This also means that functionality bundled into Umbraco, such as the usage of templates to assign different ActionResults or Views; or the ability to preview a page before publishing are lost. 

Another minor caveat is that internal links created in rich text editors will need to be parsed, as the raw value stored in the HTML would be of the form "/{localLink:umb://document/59b8e16037eb4aeabd30b13400696f42}".

Finally this setup requires multiple hosting environments and SSL certificates.
