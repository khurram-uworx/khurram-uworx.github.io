---
title: ".NET: User Secrets"
date: 2025-07-15
tags:
- .NET
comments: true
---

To switch from storing sensitive configuration data in `appsettings.json` to **User Secrets** while still using **Visual Studio to publish to Azure**, follow these step-by-step instructions:

---

### ✅ 1. **Enable User Secrets in Your Project**

In your `.csproj` file, ensure this line is included inside a `<PropertyGroup>`:

```xml
<UserSecretsId>your-project-guid-or-name</UserSecretsId>
```

If it's not there, add it manually or via Visual Studio:

* Right-click the project in Solution Explorer → **Manage User Secrets**

This will automatically create a `secrets.json` file at:

```
%APPDATA%\Microsoft\UserSecrets\<UserSecretsId>\secrets.json
```

---

### ✅ 2. **Add Your Secrets**

Add your secrets in the same structure as `appsettings.json`, for example:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "supersecret-connection-string"
  },
  "Authentication": {
    "Google": {
      "ClientId": "1234",
      "ClientSecret": "xyz"
    }
  }
}
```

You can also edit it via Visual Studio: after opening “Manage User Secrets,” paste the above into the file.

---

### ✅ 3. **Update `appsettings.json` (Optional)**

Remove the secrets from `appsettings.json` to ensure they are not checked into Git:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": ""
  },
  "Authentication": {
    "Google": {
      "ClientId": "",
      "ClientSecret": ""
    }
  }
}
```

---

### ✅ 4. **Ensure Configuration Is Loaded Properly**

In `Program.cs` or `Startup.cs`, ensure this is present (should be by default):

```csharp
builder.Configuration
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: false)
    .AddUserSecrets<Program>()
    .AddEnvironmentVariables();
```

If you're using `CreateHostBuilder` pattern:

```csharp
.ConfigureAppConfiguration((context, config) =>
{
    var env = context.HostingEnvironment;
    config.AddJsonFile("appsettings.json", optional: false)
          .AddUserSecrets<Program>()
          .AddEnvironmentVariables();
})
```

---

### ✅ 5. **Configure Azure App Service (Production Settings)**

User Secrets are only available in development. To run in **Azure**, use **App Service Configuration Settings**:

1. In Azure Portal:

   * Go to your App Service → **Configuration** (under Settings)
   * Add the following under **Application settings**:

     * `ConnectionStrings__DefaultConnection` = `supersecret-connection-string`
     * `Authentication__Google__ClientId` = `1234`
     * `Authentication__Google__ClientSecret` = `xyz`

> Note: use double underscores `__` to represent nested keys.

---

### ✅ 6. **Publish From Visual Studio**

Continue publishing as you do:

* Right-click project → **Publish**
* Pick Azure App Service
* Publish as usual

Secrets from **User Secrets** will be used during **local development**, and Azure App Settings will take over in **production**.

---

### ✅ 7. **Verify No Secrets in Git**

Make sure `appsettings.json` no longer contains real secrets, and `secrets.json` is never included in Git.

✅ Done. You're now using User Secrets for local development and Azure App Settings for production—all compatible with Visual Studio publishing.
