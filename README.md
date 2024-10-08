# NET-Upgrade-Docs

## Steps to upgrade a solution from FF(full framework) to .net 6+

1.  Run tests and ensure that everything is green. (Run tests after each of the following steps, fix any issues before moving to the next step)
1.  Upgrade all packages.config to ProjectReference - this will get rid of potential nuget conflicts.
1.  Remove unused dependencies from the solution.
1.  Upgrade to .net 4.8 if on an older version.
1.  Consolidate nuget packages and upgrade them to the latest if possible or whichever version gets your project to still run.
1.  Run all tests to make sure that all of them are still green.
1.  Walk up the dependency tree starting with the class library project that has no project dependencies and upgrade it to `netstandard2.0`.
1.  Upgrade the test projects last.

> If upgrading from WebForm (aspx), convert all pages to Razor (cshtml) before upgrading.
> Tools: https://apisof.net/upgrade-planner use upgrade planner to help pick suitable projects to upgrade and identify issues with upgrade.

## ASP.NET

| FullFramework                                                                                  | NetStandard/6+                                                                                                                                                 |
| ---------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CacheItemPriority                                                                              | Microsoft.Extensions.Caching.Memory                                                                                                                            |
| HttpContext.Request.Url                                                                        | Microsoft.AspNetCore.Http.Extensions.UriHelper.GetEncodedUrl/GetDisplayUrl(Request)                                                                            |
| HttpContext.Request.UserHostAddress                                                            | HttpContext.Connection.RemoteIpAddress                                                                                                                         |
| HttpActionContext.Response                                                                     | ActionExecutingContext.Result = new StatusCodeResult                                                                                                           |
| HttpRequest.InputStream                                                                        | HttpRequest.Body                                                                                                                                               |
| Url.Encode                                                                                     | Uri.EscapeDataString(someString)                                                                                                                               |
| HttpCookie                                                                                     | Microsoft.AspNetCore.Http.CookieOptions                                                                                                                        |
| Controller.ValueProvider.GetValue("action")                                                    | Context.GetRouteData().Values["action"]                                                                                                                        |
| Controller.Session["key"]                                                                      | HttpContext.Session.GetString("key")                                                                                                                           |
| HttpContext.Server.MapPath                                                                     | IWebHostEnvironment.ContentRootPath                                                                                                                            |
| `[ScriptIgnore]`                                                                               | `[JsonIgnore]`                                                                                                                                                 |
| `<script nonce="custom-method-call">`                                                          | `asp-add-nonce="true"`                                                                                                                                         |
| `[HandleError]`                                                                                | Add an error controller and configure middleware to handle it. [Link](https://learn.microsoft.com/en-us/aspnet/core/web-api/handle-errors?view=aspnetcore-8.0) |
| `Request.QueryString["ReturnUrl"]`                                                             | `Request.Query.TryGetValue("ReturnUrl")`/ `Request.Query["ReturnUrl"]`                                                                                         |
| Cache busting - Assembly.VersionInfo                                                           | `asp-append-version="true"`                                                                                                                                    |
| `[PrincipalPermission(SecurityAction.Demand, Role = "Admin")]`                                 | `[Authorize(Role="Admin")]`                                                                                                                                    |
| `Response.Cookies.Add(new HttpCookie("COOKIE_NAME") { Expires = DateTime.Now.AddYears(-1) });` | `Response.Cookies.Append("COOKIE_NAME","", new CookieOptions { Expires = DateTime.Now.AddYears(-1) });`                                                        |
| `MvcHtmlString`                                                                                | `Microsoft.AspNetCore.Html.HtmlString`                                                                                                                         |
| ` ViewDataDictionary{ { "Key", "Value" } }`                                                    | `ViewDataDictionary(ViewData){ { "Key", "Value" } }`                                                                                                           |
| `@helper DoHelpfulStuff(){// logic}`                                                           | `@ {void DoHelpfulStuff(){// logic}}`  invoke using `@{DoHelpfulStuff();}`                                                                                     |
| `Request.Headers.GetCookies()`                                                                 | `Request.Cookies`                                                                                                                                              |
| `request.ServerVariables["HTTP_X_FORWARDED_FOR"]` | `


## WebForms to Razor
| WebForms                          | Razor                                                                                                               |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `<%= .. %>`                       | `@( .. )`                                                                                                           |
| `<%`                              | `@{`  If multiple such lines exist replace the closing `%>` with `}`                                                |
| `<% if`                           | `@if`  All `<%` tags following the first `if` need to be removed. For automation keep track of braces               |
| `<% using (Html.BeginForm()) {%>` | `<form asp-action='Action_Name' controller='Controller_Name method="post">` keep track of the controller and action |
| ResolveUrl                        | Url.Content                                                                                                         |
|                                   |                                                                                                                     |

## Microsoft Libraries

| FullFramework          | NetStandard/6+           |
| ---------------------- | ------------------------ |
| System.Data.SqlClient  | Microsoft.Data.SqlClient |
| System.Web.HttpUtility | System.Net.WebUtility    |

## EF Upgrade

#### Entity Configuration

| EF                                                           | EF Core                                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------------ |
| EntityTypeConfiguration                                      | IEntityTypeConfiguration                                           |
| HasDatabaseGeneratedOption(DatabaseGeneratedOption.Identity) | DatabaseGeneratedOption.Identity                                   |
| HasOptional()                                                | Not required, IsRequired(false) is an option                       |
| HasOptional(x => x.PropertyName)                             | Modify the underlying property to be `virtual`                     |
| dbContext.Entity.AddOrUpdate()                               | dbContext.Entity.Update()                                          |
| dbContext.Configuration.AutoDetectChangesEnabled             | dbContext.ChangeTracker.AutoDetectChangesEnabled                   |
| dbContext.Configuration.ProxyCreationEnabled                 | dbContext.ChangeTracker.LazyLoadingEnabled                         |
| Configuration HasRequired(p=> p.Property)                    | builder.HasOne(p=>p.Property).WithMany(x=>x.property).IsRequired() |
| Configuration WillCascadeOnDelete(true)                      | OnDelete(DeleteBehavior.Cascade)                                   |
| Configuration WillCascadeOnDelete(false)                     | OnDelete(DeleteBehavior.ClientSetNull)                             |
| ObjectNotFoundException                                      | DbUpdateException                                                  |
| Enum DataType                                                | Configuration: .HasConversion<int>()                               |

## Common Errors

#### `C# version incompatibility`

Some times you can get a C# version incompatibility between the upgraded project and the old projects. Create a file with the name `Directory.Build.props` and add the following to it

```xml
<Project>
  <PropertyGroup>
  <!-- use version number instead of latest if you want a specific version -->
    <LangVersion>latest</LangVersion>
  </PropertyGroup>
</Project>
```

#### `The certificate chain was issued by an authority that is not trusted`

https://learn.microsoft.com/en-us/troubleshoot/sql/database-engine/connect/certificate-chain-not-trusted?tabs=ole-db-driver-19

#### `CS0012: The type 'System.Object' is defined in an assembly that is not referenced. You must add a reference to assembly 'netstandard, Version=2.0.0.0, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51'.`

Add the following to Web.config

```xml
<!-- targetFramework needs to be atleast 4.7.1 -->
<compilation debug="true" targetFramework="4.8">
  <assemblies>
  <add assembly="netstandard, Version=2.0.0.0, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51"/>
  </assemblies>
</compilation>
```

**OR**  
Add the following to the csproj of the asp.net project

```xml
<Reference Include="netstandard" />
```


## Side by Side upgrade
1. Create a `wwwroot` folder and copy all static assets there.
1. If any React/Angular project exist their dist should end up in the `wwwroot` folder as well.


### Useful RegEx patterns
1. Context.Session["key"] to HttpContext.Session.GetString("key")
   1. Find and replace Context with HttpContext and then use this for the remaining transformation
      1. Find: `HttpContext\.Session\[(?<key>.*)\]`
      2. Replace: `HttpContext.Session.GetString(${key})`

### Dependency Injection
 . Dependency resolution in controller without updating the controller: `HttpContext.RequestServices.GetService<YOUR_DEPENDENCY>();`


### Identity 
```csharp
await HttpContext.SignInAsync(
	            CookieAuthenticationDefaults.AuthenticationScheme,
	            new ClaimsPrincipal(CLAIMS_IDENTITY),
	            new AuthenticationProperties { IsPersistent = false });
```

#### POSTing form using JS

**OLD** 
```cshtml
<form id="FORM_ID"  method="post" action="@Url.Content("~/ACTION_NAME/" + ViewBag.YOUR_DATA)" />
```
**NEW**

```cshtml
<form id="FORM_ID" method="post" asp-route="@Url.Content("~/ACTION_NAME/" + ViewBag.YOUR_DATA)"  />
```

If not posting a query string or posting to a different controller 
```cshtml
<form id="FORM_ID" method="post" asp-action="ACTION_NAMe" asp-controller="CONTROLLER_NAME" />
```


