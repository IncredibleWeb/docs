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
## REST Conventions
`GET /users` - Retrieves a list of users
`GET /users/36` - Retrieves a user with UID 36
`GET /users/36/orders` - Retrieves a list of orders made by user with UID 36
`GET /users/36/orders/4` - Retrieves order with UID 4 made by user UID 36
`GET /users/?name=john-doe` - Retrieves user John Doe by name
`GET /users/?name=john-doe/orders` - Retrieves orders made by user John Doe

`POST /users` - Creates a new user
`POST /users/36/orders` - Creates a new order belong to user with UID 36

`PUT /users/36` - Updates user with UID 36 (**Note that `PUT` will replace the entire object**)

`PATCH /users/36` - Partially updates user with UID 36

`DELETE /users/36` - Deletes user with UID 36

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
}
```
- Install UmbracoFileSystemProviders.Azure to store files on Azure rather than locally on each machine.
`PM> Install-Package UmbracoFileSystemProviders.Azure -IncludePrerelease`
After installation, navigate to the file Config/FileProviders.config and update the storage connection string to the Azure storage account.

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
Nested document types have hierarchy in the backoffice and that makes it easier to see the structure. In order to create a child DocType a permission should be set on the Parent DocType. This can be done by going to the Parent DocType page and click on Permissions. You can choose the childs of it under that tab.

Example: Banner Folder (DocType) -> Banner (DocType)

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
- Due to the changes made to the routing, Umbraco will not be synced.
- Templates will not be synced to Umbraco
- Most of the functionality of Umbraco like Preview, Grid Editor.. etc functionality will be lost.
