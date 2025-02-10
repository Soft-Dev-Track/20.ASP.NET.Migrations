# 20.ASP.NET : Migrations
Now that we have defined our entities and database context, it's time to generate migrations and set up our database structure. 
Migrations allow us to create, modify, and update the database schema based on our entity models.

Before starting, We need to configure the database connection and enable migrations.

## 1. Configuring the Database Connection
First, we need to define our connection string in the `appsettings.json` file. This string tells Entity Framework how to connect to the database.

### Update appsettings.json
Add the following configuration inside the `ConnectionStrings` section:

```json
"ConnectionStrings": {
  "Connection": "Server=(localdb)\\mssqllocaldb;Database=YourProjectName;Trusted_Connection=True;"
}
```

This configuration specifies:

- **Server**: We are using localdb, a lightweight SQL Server instance for development.
- **Database**: The database name is `YourProjectName`.
- **Trusted_Connection=True**: Enables Windows authentication instead of requiring credentials.

*Note*: If you're using a different database provider (e.g., MySQL, PostgreSQL), you will need to modify this connection string accordingly.

## 2. Registering the Database Context in Dependency Injection
To ensure that our application correctly interacts with the database, we need to register our `DbContext` in the dependency injection (DI) container inside `Program.cs`.

### Update Program.cs

```csharp
public static void Main(string[] args)
{
     // ...
    
    // We stock the connection string here
    string? context = builder.Configuration.GetConnectionString("Connection");

    // We create here the service
    builder.Services.AddDbContext<ContextDatabase>(opt => opt.UseSqlServer(context));

    // ...
}
``` 

### Understanding Dependency Injection in ASP.NET Core
ASP.NET Core uses IoC (Inversion of Control) containers to manage dependencies, including the database context.

- The DI container allows us to register services (such as DbContext), which can then be injected wherever needed in the application.
- In the above code, we register ContextUser and configure it to use SQL Server with the connection string retrieved from appsettings.json.
- This ensures that every time the application starts, the database connection is properly initialized.

However, not all services should have the same lifecycle. ASP.NET Core provides three different lifecycles for dependency injection:
 
|Lifecyle Type | Instance Scope | Best Use Cases|
|--------------|----------------|---------------|
|Transient | A new instance is created every time the service is requested. |Lightweight, short-lived services that do not maintain state.|
|Scoped|	A new instance is created once per request (HTTP request scope). | Services that handle database transactions or request-specific data.|
|Singleton|A single instance is created and shared across the application lifetime.	| Services that need to maintain state globally, such as caching, logging, or configuration.|


### General Best Practices

- Use **Transient** for simple, lightweight operations that don’t store state.
- Use **Scoped** for database operations (`DbContext`) and request-based services.
- Use **Singleton** for shared global state, but be mindful of concurrency issues.

## 3. Check if this works well
- Go to your `Tools` > `NuGet Package Manager` > `NuGet Package Console` 
- Type : `script-dbcontext`

And this will generate this kind of file:

```sql
CREATE TABLE [Addresses] (
    [IdUser] int NOT NULL IDENTITY,
    [StreetNumber] int NOT NULL,
    [Street] nvarchar(10) NOT NULL,
    [ZipCode] int NOT NULL,
    [Town] nvarchar(10) NOT NULL,
    [Country] nvarchar(10) NOT NULL,
    CONSTRAINT [PK_Addresses] PRIMARY KEY ([IdUser])
);
GO


CREATE TABLE [Users] (
    [Id] int NOT NULL IDENTITY,
    [FirstName] nvarchar(10) NOT NULL,
    [LastName] nvarchar(10) NULL,
    [Email] nvarchar(20) NOT NULL,
    CONSTRAINT [PK_Users] PRIMARY KEY ([Id])
);
GO

``` 

## 4. Steps to Create and Manage Migrations
`Entity Framework` provides several commands to create, apply, and manage migrations. Below is a step-by-step guide on how to use them effectively.

### 1.Generate a New Migration
To create a migration, run the following command in the `Package Manager Console` (PMC) of Visual Studio:

```powershell
Add-Migration Base -OutputDir Data/Migrations
```

- **Add-Migration Base** : Creates a new migration named "Base" (you can replace "Base" with any meaningful name).
- **OutputDir Data/Migrations** : Specifies the directory where the migration files will be stored.

This command generates a new C# migration file that contains SQL commands to create or modify tables based on your entity definitions.

### 2. Check Existing Migrations
To see a list of all migrations that have been created in your project, use:

```powershell
Get-Migration
```

- It allows you to track all migrations that have been added.
- If you need to roll back to a specific migration, you can find its name here.

### 3. Apply the Migration and Update the Database

```powershell
update-Database
```

- Applies the latest migration to the database.
- If the database does not exist, Entity Framework will create it automatically.
- If new migrations exist, it executes SQL scripts to update the schema.

### 4. Apply a Specific Migration
If you want to apply a specific migration instead of the latest one, use:

```powershell
update-Database MigrationName
```

- This is useful when you need to roll back to a previous database state or apply a migration step-by-step.

### 5. Remove the Last Migration
If you made a mistake or need to modify your last migration before applying it, you can remove it using:

```powershell
Remove-Migration
```

## 5. Best Practices for Working with Migrations

-  Always check the SQL script before applying a migration (`script-dbcontext`).
- Name migrations meaningfully (e.g., `AddUsersTable` instead of `Base`).
- Apply migrations step-by-step in production instead of all at once.
- Use `Remove-Migration` before applying if you made a mistake.
- Keep track of your migration history using `Get-Migration`.