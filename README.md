# NET-Upgrade-Docs

## Steps to upgrade a solution from FF(full framework) to .net 6+  
 1. Run tests and ensure that everything is green.
 1. Upgrade all project.config to ProjectReference - this will get rid of potentially nuget conflicts.  
 1. Consolidate nuget packages and upgrade them to the latest if possible or whichever version gets your project to still run. 
 1. Run all tests to make sure that all of them are still green.
 1. Walk up the dependency tree starting with the class library project that has no project dependencies and upgrade it to `netstandard2.0`. 
 1. **Run tests after each step** to ensure that everything is still in working order. Fix issues before moving to the next step. 
 1. Upgrade the test projects last.

## EF Upgrade 
### 1. Entity Configuration

| EF | EF Core |
|-----|-----|
|EntityTypeConfiguration| IEntityTypeConfiguration|
|HasDatabaseGeneratedOption(DatabaseGeneratedOption.Identity)|DatabaseGeneratedOption.Identity|
|HasOptional() | Not required, IsRequired(false) is an option|
|HasOptional(x => x.PropertyName)| Modify the underlying property to be `virtual`|
|dbContext.Entity.AddOrUpdate()| dbContext.Entity.Update()|
|dbContext.Configuration.AutoDetectChangesEnabled|dbContext.ChangeTracker.AutoDetectChangesEnabled |
|dbContext.Configuration.ProxyCreationEnabled|dbContext.ChangeTracker.LazyLoadingEnabled|
|Configuration HasRequired(p=> p.Property) | builder.HasOne(p=>p.Property).WithMany(x=>x.property).IsRequired()|
|Configuration WillCascadeOnDelete(true) | OnDelete(DeleteBehavior.Cascade)|
|Configuration WillCascadeOnDelete(false) | OnDelete(DeleteBehavior.ClientSetNull)|
| ObjectNotFoundException | DbUpdateException|
| Enum DataType | Configuration: .HasConversion<int>()|


## ASP.NET
|FullFramework| NetStandard/6+|  
|-----|-----|  
|CacheItemPriority| Microsoft.Extensions.Caching.Memory| 
|HttpContext.Request.Url| Microsoft.AspNetCore.Http.Extensions.UriHelper.GetEncodedUrl/GetDisplayUrl(Request)|
|HttpContext.Request.UserHostAddress| HttpContext.Connection.RemoteIpAddress|  
|HttpActionContext.Response| ActionExecutingContext.Result = new StatusCodeResult|
| HttpRequest.InputStream | HttpRequest.Body|
|Url.Encode|Uri.EscapeDataString(someString)|  
|Controller.ValueProvider.GetValue("action")|Context.GetRouteData().Values["action"]|
 
