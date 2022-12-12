# NET-Upgrade-Docs


## EF Upgrade 
### 1. Entity Confguration

| EF | EF Core |
|-----|-----|
|EntityTypeConfiguration| IEntityTypeConfiguration|
|HasDatabaseGeneratedOption(DatabaseGeneratedOption.Identity)|DatabaseGeneratedOption.Identity|
|HasOptional() | Not required, IsRequired(false) is an option|
|HasOptional(x => x.PropertyName)| Modify the underlying proprty to be `virtual`|
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
  
