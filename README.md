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
