Il codice del database di identità generato richiede [Entity Framework Core migrazioni](/ef/core/managing-schemas/migrations/). Creare una migrazione e aggiornare il database. Ad esempio, eseguire i comandi seguenti:

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Nella **console di gestione pacchetti**di Visual Studio:

```powershell
Install-Package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
Add-Migration CreateIdentitySchema
Update-Database
```

# <a name="net-core-cli"></a>[Interfaccia della riga di comando di .NET Core](#tab/netcore-cli)

```dotnetcli
dotnet ef migrations add CreateIdentitySchema
dotnet ef database update
```

---

Il parametro del nome "CreateIdentitySchema" per il comando `Add-Migration` è arbitrario. `"CreateIdentitySchema"` descrive la migrazione.
