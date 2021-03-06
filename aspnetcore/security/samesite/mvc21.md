---
title: Esempio di ASP.NET Core cookie navigava sullostesso sito 2,1 MVC
author: rick-anderson
description: Esempio di ASP.NET Core cookie navigava sullostesso sito 2,1 MVC
monikerRange: '>= aspnetcore-2.1 < aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 12/03/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/samesite/mvc21
ms.openlocfilehash: 6a53c0d3c0a314c4137f071cf50062182b654658
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 05/04/2020
ms.locfileid: "82777306"
---
# <a name="aspnet-core-21-mvc-samesite-cookie-sample"></a>Esempio di ASP.NET Core cookie navigava sullostesso sito 2,1 MVC

ASP.NET Core 2,1 dispone del supporto incorporato per l'attributo [navigava sullostesso sito](https://www.owasp.org/index.php/SameSite) , ma è stato scritto nello standard originale. Il [comportamento con patch](https://github.com/dotnet/aspnetcore/issues/8212) ha modificato il significato `SameSite.None` di per emettere l'attributo navigava sullostesso sito con un valore `None`di, anziché non creare il valore. Se non si desidera creare il valore, è possibile impostare la `SameSite` proprietà su un cookie su-1.

## <a name="writing-the-samesite-attribute"></a><a name="sampleCode"></a>Scrittura dell'attributo navigava sullostesso sito

Di seguito è riportato un esempio di come scrivere un attributo navigava sullostesso sito in un cookie:

```c#
var cookieOptions = new CookieOptions
{
    // Set the secure flag, which Chrome's changes will require for SameSite none.
    // Note this will also require you to be running on HTTPS
    Secure = true,

    // Set the cookie to HTTP only which is good practice unless you really do need
    // to access it client side in scripts.
    HttpOnly = true,

    // Add the SameSite attribute, this will emit the attribute with a value of none.
    // To not emit the attribute at all set the SameSite property to (SameSiteMode)(-1).
    SameSite = SameSiteMode.None
};

// Add the cookie to the response cookie collection
Response.Cookies.Append(CookieName, "cookieValue", cookieOptions);
```

## <a name="setting-cookie-authentication-and-session-state-cookies"></a>Impostazione dei cookie di stato della sessione e dell'autenticazione dei cookie

Autenticazione dei cookie, stato sessione e [vari altri componenti](https://docs.microsoft.com/aspnet/core/security/samesite?view=aspnetcore-2.1) impostare le opzioni navigava sullostesso sito tramite opzioni cookie, ad esempio

```c#
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.Cookie.SameSite = SameSiteMode.None;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.IsEssential = true;
    });

services.AddSession(options =>
{
    options.Cookie.SameSite = SameSiteMode.None;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.IsEssential = true;
});
```

Nel codice precedente, sia l'autenticazione dei cookie che lo stato della sessione impostano il relativo attributo navigava sullostesso sito su `None`, `None` creando l'attributo con un valore e impostando anche l'attributo Secure su true.

### <a name="run-the-sample"></a>Eseguire l'esempio

Se si esegue il [progetto di esempio](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC), caricare il debugger del browser nella pagina iniziale e usarlo per visualizzare la raccolta di cookie per il sito. A tale scopo, in Edge e Chrome `F12` premere quindi selezionare `Application` la scheda e fare clic sull'URL del `Cookies` sito sotto l' `Storage` opzione nella sezione.

![Elenco dei cookie del debugger del browser](BrowserDebugger.png)

È possibile osservare dall'immagine precedente che il cookie creato dall'esempio quando si fa clic sul pulsante "crea cookie navigava sullostesso sito" ha un valore di `Lax`attributo navigava sullostesso sito, che corrisponde al valore impostato nel codice di [esempio](#sampleCode).

## <a name="intercepting-cookies"></a><a name="interception"></a>Intercettazione di cookie

Per intercettare i cookie, per modificare il valore None in base al relativo supporto nell'agente browser dell'utente, è necessario usare `CookiePolicy` il middleware. Questa deve essere inserita nella pipeline di richieste HTTP **prima** di tutti i componenti che scrivono cookie e `ConfigureServices()`configurati in.

Per inserirlo nell'uso `app.UseCookiePolicy()` della pipeline nel `Configure(IApplicationBuilder, IHostingEnvironment)` metodo in [Startup.cs](https://github.com/blowdart/AspNetSameSiteSamples/blob/master/AspNetCore21MVC/Startup.cs). Ad esempio:

```c#
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
       app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseAuthentication();
    app.UseSession();

    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });
}
```

Quindi, nel `ConfigureServices(IServiceCollection services)` configurare i criteri dei cookie per chiamare una classe helper quando i cookie vengono aggiunti o eliminati. Ad esempio:

```c#
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<CookiePolicyOptions>(options =>
    {
        options.CheckConsentNeeded = context => true;
        options.MinimumSameSitePolicy = SameSiteMode.None;
        options.OnAppendCookie = cookieContext =>
            CheckSameSite(cookieContext.Context, cookieContext.CookieOptions);
        options.OnDeleteCookie = cookieContext =>
            CheckSameSite(cookieContext.Context, cookieContext.CookieOptions);
    });
}

private void CheckSameSite(HttpContext httpContext, CookieOptions options)
{
    if (options.SameSite == SameSiteMode.None)
    {
        var userAgent = httpContext.Request.Headers["User-Agent"].ToString();
        if (SameSite.BrowserDetection.DisallowsSameSiteNone(userAgent))
        {
            options.SameSite = (SameSiteMode)(-1);
        }
    }
}
```

Funzione `CheckSameSite(HttpContext, CookieOptions)`Helper:

* Viene chiamato quando i cookie vengono aggiunti alla richiesta o eliminati dalla richiesta.
* Verifica se la `SameSite` proprietà è impostata su `None`.
* Se `SameSite` è impostato su `None` e l'agente utente corrente non supporta il valore dell'attributo None. Il controllo viene eseguito usando la classe [SameSiteSupport](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/samesite/sample/snippets/SameSiteSupport.cs) :
  * Imposta `SameSite` per non creare il valore impostando la proprietà su`(SameSiteMode)(-1)`

## <a name="targeting-net-framework"></a>.NET Framework di destinazione

ASP.NET Core e System. Web (ASP.NET classico) hanno implementazioni indipendenti di navigava sullostesso sito. Le patch di navigava sullostesso sito KB per .NET Framework non sono necessarie se si usa ASP.NET Core e non viene applicato il requisito di versione minima del Framework System. Web navigava sullostesso sito (.NET 4.7.2) a ASP.NET Core.

ASP.NET Core su .NET richiede l'aggiornamento delle dipendenze dei pacchetti NuGet per ottenere le correzioni appropriate.

Per ottenere le modifiche ASP.NET Core per .NET Framework assicurarsi di disporre di un riferimento diretto alle versioni e ai pacchetti con patch (2.1.14 o versioni successive 2,1).

```xml
<PackageReference Include="Microsoft.Net.Http.Headers" Version="2.1.14" />
<PackageReference Include="Microsoft.AspNetCore.CookiePolicy" Version="2.1.14" />
```

### <a name="more-information"></a>Altre informazioni
 
[Aggiornamenti](https://www.chromium.org/updates/same-site)
di Chrome[ASP.NET Core documentazione](https://docs.microsoft.com/aspnet/core/security/samesite?view=aspnetcore-2.1)
di navigava sullostesso sito[ASP.NET Core 2,1 annuncio di modifica navigava sullostesso sito](https://github.com/dotnet/aspnetcore/issues/8212)