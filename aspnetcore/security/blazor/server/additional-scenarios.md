---
Titolo:' ASP.NET Core Blazor Server scenari di sicurezza aggiuntivi ' autore: Descrizione:' informazioni su come configurare il Blazor Server per altri scenari di sicurezza .'
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

---
# <a name="aspnet-core-blazor-server-additional-security-scenarios"></a>BlazorScenari di sicurezza aggiuntivi di ASP.NET Core Server

Di [Javier Calvarro Nelson](https://github.com/javiercn)

## <a name="pass-tokens-to-a-blazor-server-app"></a>Passare i token a un' Blazor app Server

I token disponibili all'esterno dei Razor componenti di un' Blazor app Server possono essere passati ai componenti di con l'approccio descritto in questa sezione. Per il codice di esempio, incluso un `Startup.ConfigureServices` esempio completo, vedere il [passaggio di token a un' Blazor applicazione lato server](https://github.com/javiercn/blazor-server-aad-sample).

Autenticare l' Blazor app Server come si farebbe con un' Razor applicazione MVC o pagine normali. Eseguire il provisioning e salvare i token nel cookie di autenticazione. Ad esempio:

```csharp
using Microsoft.AspNetCore.Authentication.OpenIdConnect;

...

services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
{
    options.ResponseType = "code";
    options.SaveTokens = true;

    options.Scope.Add("offline_access");
    options.Scope.Add("{SCOPE}");
    options.Resource = "{RESOURCE}";
});
```

Definire una classe per passare lo stato iniziale dell'app con i token di accesso e di aggiornamento:

```csharp
public class InitialApplicationState
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

Definire un servizio del provider di token con **ambito** che può essere usato nell' Blazor app per risolvere i token da [inserimento delle dipendenze](xref:blazor/dependency-injection):

```csharp
public class TokenProvider
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

In `Startup.ConfigureServices` aggiungere servizi per:

* `IHttpClientFactory`
* `TokenProvider`

```csharp
services.AddHttpClient();
services.AddScoped<TokenProvider>();
```

Nel file *_Host. cshtml* creare un'istanza di e `InitialApplicationState` passarla come parametro all'app:

```cshtml
@using Microsoft.AspNetCore.Authentication

...

@{
    var tokens = new InitialApplicationState
    {
        AccessToken = await HttpContext.GetTokenAsync("access_token"),
        RefreshToken = await HttpContext.GetTokenAsync("refresh_token")
    };
}

<app>
    <component type="typeof(App)" param-InitialState="tokens" 
        render-mode="ServerPrerendered" />
</app>
```

Nel `App` componente (*app. Razor*) risolvere il servizio e inizializzarlo con i dati del parametro:

```razor
@inject TokenProvider TokenProvider

...

@code {
    [Parameter]
    public InitialApplicationState InitialState { get; set; }

    protected override Task OnInitializedAsync()
    {
        TokenProvider.AccessToken = InitialState.AccessToken;
        TokenProvider.RefreshToken = InitialState.RefreshToken;

        return base.OnInitializedAsync();
    }
}
```

Nel servizio che esegue una richiesta API protetta, inserire il provider di token e recuperare il token per chiamare l'API:

```csharp
public class WeatherForecastService
{
    private readonly TokenProvider _store;

    public WeatherForecastService(IHttpClientFactory clientFactory, 
        TokenProvider tokenProvider)
    {
        Client = clientFactory.CreateClient();
        _store = tokenProvider;
    }

    public HttpClient Client { get; }

    public async Task<WeatherForecast[]> GetForecastAsync(DateTime startDate)
    {
        var token = _store.AccessToken;
        var request = new HttpRequestMessage(HttpMethod.Get, 
            "https://localhost:5003/WeatherForecast");
        request.Headers.Add("Authorization", $"Bearer {token}");
        var response = await Client.SendAsync(request);
        response.EnsureSuccessStatusCode();

        return await response.Content.ReadAsAsync<WeatherForecast[]>();
    }
}
```

## <a name="use-open-id-connect-oidc-v20-endpoints"></a>Usa endpoint Open ID Connect (OIDC) v 2.0

I modelli e la libreria di autenticazione Blazor usano gli endpoint Open ID Connect (OIDC) v 1.0. Per usare un endpoint v 2.0, configurare l' <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority?displayProperty=nameWithType> opzione in <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> :

```csharp
services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, 
    options =>
    {
        options.Authority += "/v2.0";
    }
```

In alternativa, l'impostazione può essere eseguita nel file delle impostazioni dell'app (*appSettings. JSON*):

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/common/oauth2/v2.0/",
    ...
  }
}
```

Se l'uso di un segmento nell'autorità non è appropriato per il provider OIDC dell'app, ad esempio con i provider non AAD, impostare <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> direttamente la proprietà. Impostare la proprietà in <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> o nel file di impostazioni dell'app con la <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> chiave.

### <a name="code-changes"></a>Modifiche al codice

* L'elenco di attestazioni nel token ID cambia per gli endpoint della versione 2.0. Per ulteriori informazioni, vedere [perché eseguire l'aggiornamento alla piattaforma di identità Microsoft (v 2.0)?](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison) nella documentazione di Azure.
* Poiché le risorse vengono specificate negli URI di ambito per gli endpoint v 2.0, rimuovere l' <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Resource?displayProperty=nameWithType> impostazione della proprietà in <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> :

  ```csharp
  services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options => 
      {
          ...
          options.Resource = "...";    // REMOVE THIS LINE
          ...
      }
      ```

  For more information, see [Scopes, not resources](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison#scopes-not-resources) in the Azure documentation.

### App ID URI

* When using v2.0 endpoints, APIs define an *App ID URI*, which is meant to represent a unique identifier for the API.
* All scopes include the App ID URI as a prefix, and v2.0 endpoints emit access tokens with the App ID URI as the audience.
* When using V2.0 endpoints, the client ID configured in the Server API changes from the API Application ID (Client ID) to the App ID URI.

*appsettings.json*:

```json
{
  "AzureAd": {
    ...
    "ClientId": "https://{TENANT}.onmicrosoft.com/{APP NAME}"
    ...
  }
}
```

È possibile trovare l'URI ID app da usare nella descrizione della registrazione dell'app provider OIDC.
