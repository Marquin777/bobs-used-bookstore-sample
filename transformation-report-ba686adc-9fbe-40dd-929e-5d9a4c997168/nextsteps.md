# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The solution has been transformed with no build errors across all projects:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

Since no build errors were detected, the transformation to cross-platform .NET appears to have completed successfully. The following steps outline how to validate, test, and deploy the solution.

---

## 1. Restore and Build the Solution

Run the following commands from the root of the solution to confirm a clean restore and build:

```bash
dotnet restore
dotnet build --configuration Release
```

Ensure there are no warnings that could indicate compatibility issues, such as deprecated APIs or platform-specific code paths.

---

## 2. Run Unit Tests

Execute the test project to confirm all existing tests pass under the new runtime:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output carefully. Any failing tests may indicate behavioral differences introduced by the migration to cross-platform .NET.

---

## 3. Verify Runtime Behavior of `Bookstore.Data`

Since `Bookstore.Data` likely handles database access, confirm the following:

- The correct database provider NuGet package is referenced (e.g., `Microsoft.EntityFrameworkCore.SqlServer`, `Npgsql.EntityFrameworkCore.PostgreSQL`, or `Microsoft.EntityFrameworkCore.Sqlite`).
- Connection strings in configuration files (`appsettings.json`) are valid and accessible in the target environment.
- Run any available database migrations to ensure the schema is up to date:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

---

## 4. Validate the Web Application Locally

Run the web application locally to confirm it starts and functions correctly:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following:

- The application starts without runtime exceptions.
- All routes and pages load as expected.
- Any authentication, session, or middleware configuration behaves correctly under ASP.NET Core.

---

## 5. Review `Bookstore.Cdk` Configuration

If `Bookstore.Cdk` is an AWS CDK project for infrastructure definition, verify the following:

- The CDK project targets a compatible .NET version.
- All stack definitions reference the correct resource configurations for the target environment.
- Run a CDK synthesis to validate the infrastructure output without deploying:

```bash
cd app/Bookstore.Cdk
cdk synth
```

Review the synthesized output for any configuration issues before proceeding to deployment.

---

## 6. Check for Platform-Specific Code

Even without build errors, review the codebase for any remaining platform-specific patterns that may cause issues at runtime on non-Windows environments:

- Windows registry access (`Microsoft.Win32.Registry`)
- Windows-specific file path separators (use `Path.Combine` instead of hardcoded `\`)
- `System.Drawing` usage (not fully supported cross-platform without additional packages)
- Any P/Invoke calls targeting Windows-only native libraries

---

## 7. Deploy the Application

Once local validation is complete, publish the application for the target runtime:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

Copy the contents of the `./publish` directory to the target hosting environment and configure the web server (e.g., IIS, Nginx, or Kestrel as a standalone server) accordingly.