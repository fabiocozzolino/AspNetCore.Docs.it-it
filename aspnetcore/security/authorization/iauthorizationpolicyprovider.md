---
title: Provider di criteri di autorizzazione personalizzati in ASP.NET Core
author: mjrousos
description: Informazioni su come usare un IAuthorizationPolicyProvider personalizzato in un'app ASP.NET Core per generare dinamicamente i criteri di autorizzazione.
ms.author: riande
ms.custom: mvc
ms.date: 11/14/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/authorization/iauthorizationpolicyprovider
ms.openlocfilehash: 1db78e5b2cea964471e4eea090f713f6af5f4740
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 05/04/2020
ms.locfileid: "82777540"
---
# <a name="custom-authorization-policy-providers-using-iauthorizationpolicyprovider-in-aspnet-core"></a>Provider di criteri di autorizzazione personalizzati che usano IAuthorizationPolicyProvider in ASP.NET Core 

Di [Mike risveglia](https://github.com/mjrousos)

In genere, quando si utilizza l' [autorizzazione basata su criteri](xref:security/authorization/policies), i `AuthorizationOptions.AddPolicy` criteri vengono registrati chiamando come parte della configurazione del servizio di autorizzazione. In alcuni scenari potrebbe non essere possibile (o auspicabile) registrare tutti i criteri di autorizzazione in questo modo. In questi casi, è possibile usare un oggetto `IAuthorizationPolicyProvider` personalizzato per controllare come vengono forniti i criteri di autorizzazione.

Esempi di scenari in cui un [IAuthorizationPolicyProvider](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationpolicyprovider) personalizzato può essere utile includono:

* Uso di un servizio esterno per fornire la valutazione dei criteri.
* Usando un'ampia gamma di criteri (ad esempio, per diversi numeri di stanza o età), non ha senso aggiungere ogni singolo criterio di autorizzazione con una `AuthorizationOptions.AddPolicy` chiamata.
* Creazione di criteri in fase di esecuzione in base alle informazioni in un'origine dati esterna, ad esempio un database, o determinazione dinamica dei requisiti di autorizzazione tramite un altro meccanismo.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/aspnetcore/tree/v3.1.3/src/Security/samples/CustomPolicyProvider) dal [repository GitHub AspNetCore](https://github.com/dotnet/AspNetCore). Scaricare il file ZIP del repository DotNet/AspNetCore. Decomprimere il file. Passare alla cartella di progetto *SRC/Security/Samples/CustomPolicyProvider* .

## <a name="customize-policy-retrieval"></a>Personalizzare il recupero dei criteri

Le app ASP.NET Core usano un'implementazione dell' `IAuthorizationPolicyProvider` interfaccia per recuperare i criteri di autorizzazione. Per impostazione predefinita, [DefaultAuthorizationPolicyProvider](/dotnet/api/microsoft.aspnetcore.authorization.defaultauthorizationpolicyprovider) è registrato e usato. `DefaultAuthorizationPolicyProvider`Restituisce i criteri dall' `AuthorizationOptions` oggetto fornito in `IServiceCollection.AddAuthorization` una chiamata a.

Personalizzare questo comportamento registrando un'implementazione diversa `IAuthorizationPolicyProvider` nel contenitore di inserimento delle [dipendenze](xref:fundamentals/dependency-injection) dell'app. 

L' `IAuthorizationPolicyProvider` interfaccia contiene tre API:

* [GetPolicyAsync](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationpolicyprovider.getpolicyasync#Microsoft_AspNetCore_Authorization_IAuthorizationPolicyProvider_GetPolicyAsync_System_String_) restituisce un criterio di autorizzazione per un nome specificato.
* [GetDefaultPolicyAsync](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationpolicyprovider.getdefaultpolicyasync) restituisce i criteri di autorizzazione predefiniti (il criterio utilizzato `[Authorize]` per gli attributi senza un criterio specificato). 
* [GetFallbackPolicyAsync](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationpolicyprovider.getfallbackpolicyasync) restituisce i criteri di autorizzazione di fallback (il criterio utilizzato dal middleware di autorizzazione quando non è specificato alcun criterio). 

Implementando queste API, è possibile personalizzare il modo in cui vengono forniti i criteri di autorizzazione.

## <a name="parameterized-authorize-attribute-example"></a>Esempio di attributo di autorizzazione con parametri

Uno scenario in `IAuthorizationPolicyProvider` cui è utile è l' `[Authorize]` abilitazione di attributi personalizzati i cui requisiti dipendono da un parametro. Nella documentazione [sull'autorizzazione basata su criteri](xref:security/authorization/policies) , ad esempio, è stato usato un criterio basato sull'età ("AtLeast21") come esempio. Se le azioni del controller diverse in un'app devono essere rese disponibili agli utenti con età *diverse* , potrebbe essere utile avere molti criteri basati sull'età diversi. Anziché registrare tutti i diversi criteri basati sull'età necessari per l'applicazione, è possibile generare `AuthorizationOptions`i criteri in modo dinamico con un oggetto personalizzato `IAuthorizationPolicyProvider`. Per semplificare l'uso dei criteri, è possibile annotare le azioni con l'attributo `[MinimumAgeAuthorize(20)]`di autorizzazione personalizzato, ad esempio.

## <a name="custom-authorization-attributes"></a>Attributi di autorizzazione personalizzati

I criteri di autorizzazione sono identificati dai rispettivi nomi. L'oggetto `MinimumAgeAuthorizeAttribute` personalizzato descritto in precedenza deve eseguire il mapping degli argomenti in una stringa che può essere utilizzata per recuperare i criteri di autorizzazione corrispondenti. È possibile eseguire questa operazione derivando `AuthorizeAttribute` da e rendendo `Age` la proprietà incapsulata `AuthorizeAttribute.Policy` .

```csharp
internal class MinimumAgeAuthorizeAttribute : AuthorizeAttribute
{
    const string POLICY_PREFIX = "MinimumAge";

    public MinimumAgeAuthorizeAttribute(int age) => Age = age;

    // Get or set the Age property by manipulating the underlying Policy property
    public int Age
    {
        get
        {
            if (int.TryParse(Policy.Substring(POLICY_PREFIX.Length), out var age))
            {
                return age;
            }
            return default(int);
        }
        set
        {
            Policy = $"{POLICY_PREFIX}{value.ToString()}";
        }
    }
}
```

Questo tipo di attributo ha `Policy` una stringa basata sul prefisso hardcoded (`"MinimumAge"`) e un numero intero passato tramite il costruttore.

È possibile applicarlo alle azioni in modo analogo ad altri `Authorize` attributi, ad eccezione del fatto che accetta un Integer come parametro.

```csharp
[MinimumAgeAuthorize(10)]
public IActionResult RequiresMinimumAge10()
```

## <a name="custom-iauthorizationpolicyprovider"></a>IAuthorizationPolicyProvider personalizzato

Il personalizzato `MinimumAgeAuthorizeAttribute` rende più semplice richiedere i criteri di autorizzazione per qualsiasi età minima desiderata. Il prossimo problema da risolvere è garantire che i criteri di autorizzazione siano disponibili per tutte le epoche diverse. Questa è la posizione `IAuthorizationPolicyProvider` in cui è utile.

Quando si `MinimumAgeAuthorizationAttribute`USA, i nomi dei criteri di autorizzazione seguiranno `"MinimumAge" + Age`il modello, pertanto `IAuthorizationPolicyProvider` l'oggetto personalizzato deve generare i criteri di autorizzazione per:

* Analisi dell'età dal nome del criterio.
* Utilizzo `AuthorizationPolicyBuilder` di per creare un nuovo`AuthorizationPolicy`
* In questo e negli esempi seguenti si presuppone che l'utente venga autenticato tramite un cookie. Il `AuthorizationPolicyBuilder` deve essere costruito con almeno un nome dello schema di autorizzazione o avere sempre esito positivo. In caso contrario, non sono disponibili informazioni su come fornire un problema all'utente e verrà generata un'eccezione.
* Aggiunta di requisiti ai criteri in base all'età con `AuthorizationPolicyBuilder.AddRequirements`. In altri scenari, è possibile usare `RequireClaim`invece `RequireRole`, o `RequireUserName` .

```csharp
internal class MinimumAgePolicyProvider : IAuthorizationPolicyProvider
{
    const string POLICY_PREFIX = "MinimumAge";

    // Policies are looked up by string name, so expect 'parameters' (like age)
    // to be embedded in the policy names. This is abstracted away from developers
    // by the more strongly-typed attributes derived from AuthorizeAttribute
    // (like [MinimumAgeAuthorize()] in this sample)
    public Task<AuthorizationPolicy> GetPolicyAsync(string policyName)
    {
        if (policyName.StartsWith(POLICY_PREFIX, StringComparison.OrdinalIgnoreCase) &&
            int.TryParse(policyName.Substring(POLICY_PREFIX.Length), out var age))
        {
            var policy = new AuthorizationPolicyBuilder(CookieAuthenticationDefaults.AuthenticationScheme);
            policy.AddRequirements(new MinimumAgeRequirement(age));
            return Task.FromResult(policy.Build());
        }

        return Task.FromResult<AuthorizationPolicy>(null);
    }
}
```

## <a name="multiple-authorization-policy-providers"></a>Provider di criteri di autorizzazione multipli

Quando si utilizzano `IAuthorizationPolicyProvider` implementazioni personalizzate, tenere presente che ASP.NET Core utilizza solo un'istanza di `IAuthorizationPolicyProvider`. Se un provider personalizzato non è in grado di fornire i criteri di autorizzazione per tutti i nomi di criteri che verranno usati, deve rinviare a un provider di backup. 

Si consideri, ad esempio, un'applicazione che richiede criteri di validità personalizzati e il recupero di criteri più tradizionali basati sui ruoli. Tale app potrebbe usare un provider di criteri di autorizzazione personalizzato che:

* Tenta di analizzare i nomi dei criteri. 
* Chiama un provider di criteri diverso (ad `DefaultAuthorizationPolicyProvider`esempio) se il nome dei criteri non contiene un'età.

L'implementazione `IAuthorizationPolicyProvider` di esempio illustrata in precedenza può essere aggiornata `DefaultAuthorizationPolicyProvider` per usare il creando un provider di criteri di backup nel relativo costruttore (da usare nel caso in cui il nome dei criteri non corrisponda al modello previsto di "minime" + Age).

```csharp
private DefaultAuthorizationPolicyProvider BackupPolicyProvider { get; }

public MinimumAgePolicyProvider(IOptions<AuthorizationOptions> options)
{
    // ASP.NET Core only uses one authorization policy provider, so if the custom implementation
    // doesn't handle all policies it should fall back to an alternate provider.
    BackupPolicyProvider = new DefaultAuthorizationPolicyProvider(options);
}
```

Quindi, il `GetPolicyAsync` metodo può essere aggiornato in modo da `BackupPolicyProvider` usare anziché restituire null:

```csharp
...
return BackupPolicyProvider.GetPolicyAsync(policyName);
```

## <a name="default-policy"></a>Criteri predefiniti

Oltre a fornire i criteri di autorizzazione denominati `IAuthorizationPolicyProvider` , è necessario `GetDefaultPolicyAsync` implementare un oggetto personalizzato per fornire `[Authorize]` un criterio di autorizzazione per gli attributi senza specificare un nome di criterio.

In molti casi, questo attributo di autorizzazione richiede solo un utente autenticato, pertanto è possibile effettuare i criteri necessari con una chiamata `RequireAuthenticatedUser`a:

```csharp
public Task<AuthorizationPolicy> GetDefaultPolicyAsync() => 
    Task.FromResult(new AuthorizationPolicyBuilder(CookieAuthenticationDefaults.AuthenticationScheme).RequireAuthenticatedUser().Build());
```

Come per tutti gli aspetti di un `IAuthorizationPolicyProvider`oggetto personalizzato, è possibile personalizzarlo in base alle esigenze. In alcuni casi, può essere utile recuperare i criteri predefiniti da un fallback `IAuthorizationPolicyProvider`.

## <a name="fallback-policy"></a>Criteri di fallback

Un oggetto `IAuthorizationPolicyProvider` personalizzato può facoltativamente `GetFallbackPolicyAsync` implementare per fornire un criterio usato durante la [combinazione di criteri](/dotnet/api/microsoft.aspnetcore.authorization.authorizationpolicy.combine) e quando non vengono specificati criteri. Se `GetFallbackPolicyAsync` restituisce un criterio non null, i criteri restituiti vengono utilizzati dal middleware di autorizzazione quando non vengono specificati criteri per la richiesta.

Se non è richiesto alcun criterio di fallback, il provider `null` può restituire o rinviare al provider di fallback:

```csharp
public Task<AuthorizationPolicy> GetFallbackPolicyAsync() => 
    Task.FromResult<AuthorizationPolicy>(null);
```

## <a name="use-a-custom-iauthorizationpolicyprovider"></a>Usare un IAuthorizationPolicyProvider personalizzato

Per usare i criteri personalizzati da `IAuthorizationPolicyProvider`un, è necessario:

* Registrare i tipi `AuthorizationHandler` appropriati con l'inserimento di dipendenze (descritto in [autorizzazione basata su criteri](xref:security/authorization/policies#authorization-handlers)), come per tutti gli scenari di autorizzazione basata su criteri.
* Registrare il tipo `IAuthorizationPolicyProvider` personalizzato nella raccolta di servizi di inserimento delle dipendenze dell' `Startup.ConfigureServices`app (in) per sostituire il provider di criteri predefinito.

```csharp
services.AddSingleton<IAuthorizationPolicyProvider, MinimumAgePolicyProvider>();
```

Un esempio completo `IAuthorizationPolicyProvider` personalizzato è disponibile nel [repository GitHub DotNet/aspnetcore](https://github.com/dotnet/aspnetcore/tree/v3.1.3/src/Security/samples/CustomPolicyProvider).
