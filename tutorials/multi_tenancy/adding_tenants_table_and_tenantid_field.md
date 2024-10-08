# Adding Tenants Table and TenantId Field

We need to add a TenantId field to all tables, to isolate tenants from each other.

We first need a Tenants table. 

As MovieDB already have records, we'll define a primary tenant with ID 1, and set all existing records TenantId to it.

It's time to write a migration, actually two migrations, one for Northwind and one for Default database.

**DefaultDB_20170430_134800_MultiTenant.cs:**

```csharp
using FluentMigrator;

namespace MultiTenancy.Migrations.DefaultDB
{
    [Migration(20170430134800)]
    public class DefaultDB_20170430_134800_MultiTenant
        : AutoReversingMigration
    {
        public override void Up()
        {
            this.CreateTableWithId32("Tenants", "TenantId", s => s
                .WithColumn("TenantName").AsString(100)
                    .NotNullable());

            Insert.IntoTable("Tenants")
                .Row(new
                {
                    TenantName = "Primary Tenant"
                });

            Insert.IntoTable("Tenants")
                .Row(new
                {
                    TenantName = "Second Tenant"
                });

            Insert.IntoTable("Tenants")
                .Row(new
                {
                    TenantName = "Third Tenant"
                });

            Alter.Table("Users")
                .AddColumn("TenantId").AsInt32()
                    .NotNullable().WithDefaultValue(1);

            Alter.Table("Roles")
                .AddColumn("TenantId").AsInt32()
                    .NotNullable().WithDefaultValue(1);

            Alter.Table("Languages")
                .AddColumn("TenantId").AsInt32()
                    .NotNullable().WithDefaultValue(1);
        }
    }
}
```

I have created Tenants table in Default database where user tables are. Here we add 3 predefined tenants. We actually only need first one with ID *1*.

We didn't add TenantId column to tables like UserPermissions, UserRoles, RolePermissions etc, as they instrinsicly have TenantId information through their UserId or RoleId (as these tables already have TenantId value)

Let's write another migration for MovieDB tables to add TenantId column:

**DefaultDB_20170430_194100_MovieDB_MultiTenant.cs:**

```csharp
using FluentMigrator;

namespace MultiTenancy.Migrations.DefaultDB
{
    [Migration(20170430194100)]
    public class DefaultDB_20170430_194100_MovieDB_MultiTenant
        : AutoReversingMigration
    {
        public override void Up()
        {
            Alter.Table("Person").InSchema("mov")
                .AddColumn("TenantId").AsInt32()
                    .NotNullable().WithDefaultValue(1);

            Alter.Table("Genre").InSchema("mov")
                .AddColumn("TenantId").AsInt32()
                    .NotNullable().WithDefaultValue(1);

            Alter.Table("Movie").InSchema("mov")
                .AddColumn("TenantId").AsInt32()
                    .NotNullable().WithDefaultValue(1);
        }
    }
}
```