# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The solution build produced no errors across all five projects after transformation:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

This indicates the migration to cross-platform .NET was completed without introducing any compilation errors. The following steps outline how to validate, test, and deploy the solution.

---

## 1. Restore and Build the Solution

Run the following commands from the root of the solution to confirm a clean restore and build:

```bash
dotnet restore
dotnet build --configuration Release
```

Verify that the output shows no errors or warnings that could indicate runtime issues.

---

## 2. Run the Unit Tests

Execute the test project to confirm all existing tests pass under the new framework:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output for:
- Any failed tests
- Any skipped tests that may have been platform-specific in the original project
- Any runtime exceptions that do not surface as build errors

---

## 3. Verify Data Layer Behavior

Since `Bookstore.Data` handles persistence, confirm the following:

- Database connection strings are correctly configured for the target environment (e.g., `appsettings.json` or environment variables).
- Any Entity Framework Core migrations are up to date. Run the following to check:

```bash
dotnet ef migrations list --project app/Bookstore.Data
```

- If migrations are missing or out of sync, generate a new migration:

```bash
dotnet ef migrations add InitialMigration --project app/Bookstore.Data
dotnet ef database update --project app/Bookstore.Data
```

---

## 4. Validate the Web Application

Run the web project locally to confirm it starts and functions correctly:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following:
- The application starts without runtime exceptions.
- All routes respond as expected.
- Static assets load correctly.
- Any authentication or session middleware is functioning properly.

---

## 5. Review the CDK Project

The `Bookstore.Cdk` project likely defines infrastructure. Confirm the following:

- All AWS CDK or infrastructure dependencies are referencing the correct NuGet package versions compatible with the target .NET version.
- Run a synthesis or diff to validate the infrastructure definitions are intact:

```bash
dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj
```

---

## 6. Check for Platform-Specific Code

Even with a successful build, there may be runtime issues caused by APIs that behave differently across platforms. Search the codebase for the following:

- Use of `System.Windows` or other Windows-specific namespaces.
- File path separators hardcoded as `\` instead of using `Path.Combine` or `Path.DirectorySeparatorChar`.
- Registry access or Windows-specific interop calls.

Use the .NET Compatibility Analyzer if not already applied:

```xml
<PackageReference Include="Microsoft.DotNet.Analyzers.Compatibility" Version="0.2.12-alpha" />
```

---

## 7. Review Target Framework Monikers

Open each `.csproj` file and confirm the `<TargetFramework>` element is set to the intended version, for example:

```xml
<TargetFramework>net8.0</TargetFramework>
```

Ensure all projects are targeting a consistent and supported version of .NET.

---

## 8. Publish the Application

Once validation is complete, publish the web application:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

Verify the contents of the `./publish` directory and confirm all required runtime files and assets are present before deploying to the target environment.