---
title: Usare i cookie navigava sullostesso sito in ASP.NET Core
author: rick-anderson
description: Informazioni su come usare per navigava sullostesso sito i cookie in ASP.NET Core
ms.author: riande
ms.custom: mvc
ms.date: 12/03/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
- Electron
uid: security/samesite
ms.openlocfilehash: 43d5a3dbc5e202688e006355e0b105a86d721460
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 05/04/2020
ms.locfileid: "82775102"
---
# <a name="work-with-samesite-cookies-in-aspnet-core"></a>Usare i cookie navigava sullostesso sito in ASP.NET Core

Autore: [Rick Anderson](https://twitter.com/RickAndMSFT)

Navigava sullostesso sito è uno standard [IETF](https://ietf.org/about/) Draft progettato per garantire una protezione contro gli attacchi di richiesta intersito falsificazione (CSRF). Originariamente redatto in [2016](https://tools.ietf.org/html/draft-west-first-party-cookies-07), il Draft standard è stato aggiornato in [2019](https://tools.ietf.org/html/draft-west-cookie-incrementalism-00). Lo standard aggiornato non è compatibile con le versioni precedenti dello standard precedente, con le differenze più evidenti tra quelle riportate di seguito:

* I cookie senza intestazione navigava sullostesso sito vengono considerati `SameSite=Lax` come per impostazione predefinita.
* `SameSite=None`deve essere usato per consentire l'uso di cookie tra siti.
* I cookie che `SameSite=None` asseriscono anche devono `Secure`essere contrassegnati come.
* Le applicazioni che [`<iframe>`](https://developer.mozilla.org/docs/Web/HTML/Element/iframe) utilizzano potrebbero riscontrare `sameSite=Lax` problemi `sameSite=Strict` con i `<iframe>` cookie di o perché vengono considerati scenari tra siti.
* Il valore `SameSite=None` non è consentito dallo [standard 2016](https://tools.ietf.org/html/draft-west-first-party-cookies-07) e fa in modo che alcune implementazioni considerino tali `SameSite=Strict`cookie come. Vedere [supporto di browser meno recenti](#sob) in questo documento.

L' `SameSite=Lax` impostazione funziona per la maggior parte dei cookie dell'applicazione. Alcune forme di autenticazione come [OpenID Connect](https://openid.net/connect/) (OIDC) e l'impostazione predefinita di [WS-Federation](https://auth0.com/docs/protocols/ws-fed) sono i reindirizzamenti basati su post. I reindirizzamenti basati su POST attivano le protezioni del browser navigava sullostesso sito, pertanto navigava sullostesso sito è disabilitato per questi componenti. La maggior parte degli accessi [OAuth](https://oauth.net/) non è interessata a causa di differenze nel modo in cui la richiesta fluisce.

Ogni componente ASP.NET Core che emette cookie deve decidere se navigava sullostesso sito è appropriato.

## <a name="samesite-test-sample-code"></a>Codice di esempio di test di navigava sullostesso sito

 ::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

È possibile scaricare e testare gli esempi seguenti:

| Esempio               | Document |
| ----------------- | ------------ |
| [MVC .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC)  | <xref:security/samesite/mvc21> |
| [Razor Pages .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21RazorPages)  | <xref:security/samesite/rp21> |

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

È possibile scaricare e testare l'esempio seguente:


| Esempio               | Document |
| ----------------- | ------------ |
| [Razor Pages .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore31RazorPages)  | <xref:security/samesite/rp31> |

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

## <a name="net-core-support-for-the-samesite-attribute"></a>Supporto di .NET Core per l'attributo navigava sullostesso sito

.NET Core 2,2 supporta lo standard 2019 Draft per navigava sullostesso sito dal rilascio degli aggiornamenti nel 2019 dicembre. Gli sviluppatori sono in grado di controllare a livello di codice il valore dell'attributo `HttpCookie.SameSite` navigava sullostesso sito usando la proprietà. Impostando `SameSite` la proprietà su Strict, LAX o None, i valori vengono scritti in rete con il cookie. L'impostazione di un valore uguale a (SameSiteMode) (-1) indica che non è necessario includere nella rete alcun attributo navigava sullostesso sito con il cookie

[!code-csharp[](samesite/snippets/Privacy.cshtml.cs?name=snippet)]

.NET Core 3,0 supporta i valori navigava sullostesso sito aggiornati e aggiunge un valore enum aggiuntivo `SameSiteMode.Unspecified` all' `SameSiteMode` enumerazione.
Questo nuovo valore indica che non è necessario inviare navigava sullostesso sito con il cookie.

::: moniker-end

::: moniker range="= aspnetcore-2.1"

## <a name="december-patch-behavior-changes"></a>Modifiche al comportamento della patch di dicembre

La modifica del comportamento specifico per .NET Framework e .NET Core 2,1 è il `SameSite` modo in cui la `None` proprietà interpreta il valore. Prima della patch, il valore `None` significava "non emettere l'attributo", dopo la patch significa "creare l'attributo con un valore di `None`". Dopo la patch, `SameSite` un valore `(SameSiteMode)(-1)` di causa la mancata creazione dell'attributo.

Il valore predefinito di navigava sullostesso sito per l'autenticazione basata su form e i cookie `None` di `Lax`stato della sessione è stato modificato da a.

::: moniker-end

## <a name="api-usage-with-samesite"></a>Utilizzo delle API con navigava sullostesso sito

[HttpContext. Response. cookies. Append](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*) viene impostato `Unspecified`sul valore predefinito, ovvero nessun attributo navigava sullostesso sito aggiunto al cookie e il client utilizzerà il comportamento predefinito (LAX per i nuovi browser, None per quelli precedenti). Il codice seguente illustra come modificare il valore di navigava sullostesso sito del cookie `SameSiteMode.Lax`in:

[!code-csharp[](samesite/sample/Pages/Index.cshtml.cs?name=snippet)]

Tutti i componenti ASP.NET Core che emettono cookie sostituiscono i valori predefiniti precedenti con le impostazioni appropriate per gli scenari. L'override dei valori predefiniti precedenti non è stato modificato.

| Componente | cookie | Predefinito |
| ------------- | ------------- |
| <xref:Microsoft.AspNetCore.Http.CookieBuilder> | <xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite> | `Unspecified` |
| <xref:Microsoft.AspNetCore.Http.HttpContext.Session>  | [SessionOptions. cookie](xref:Microsoft.AspNetCore.Builder.SessionOptions.Cookie) |`Lax` |
| <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.CookieTempDataProvider>  | [CookieTempDataProviderOptions. cookie](xref:Microsoft.AspNetCore.Mvc.CookieTempDataProviderOptions.Cookie) | `Lax` |
| <xref:Microsoft.AspNetCore.Antiforgery.IAntiforgery> | [AntiforgeryOptions. cookie](xref:Microsoft.AspNetCore.Antiforgery.AntiforgeryOptions.Cookie)| `Strict` |
| [Autenticazione cookie](xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*) | [CookieAuthenticationOptions. cookie](xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.CookieName) | `Lax` |
| <xref:Microsoft.Extensions.DependencyInjection.TwitterExtensions.AddTwitter*> | [TwitterOptions.StateCookie](xref:Microsoft.AspNetCore.Authentication.Twitter.TwitterOptions.StateCookie) | `Lax`  |
| <xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationHandler`1> | [RemoteAuthenticationOptions.CorrelationCookie](xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.CorrelationCookie)  | `None` |
| <xref:Microsoft.Extensions.DependencyInjection.OpenIdConnectExtensions.AddOpenIdConnect*> | [OpenIdConnectOptions.NonceCookie](xref:Microsoft.AspNetCore.Authentication.OpenIdConnect.OpenIdConnectOptions.NonceCookie)| `None` |
| [HttpContext. Response. cookies. Append](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*) | <xref:Microsoft.AspNetCore.Http.CookieOptions> | `Unspecified` |

::: moniker range=">= aspnetcore-3.1"

ASP.NET Core 3,1 e versioni successive fornisce il supporto navigava sullostesso sito seguente:

* Ridefinisce il comportamento di `SameSiteMode.None` da emettere`SameSite=None`
* Aggiunge un nuovo valore `SameSiteMode.Unspecified` per omettere l'attributo navigava sullostesso sito.
* Per `Unspecified`impostazione predefinita, tutte le API cookie. Alcuni componenti che usano cookie impostano valori più specifici per gli scenari. Vedere la tabella precedente per gli esempi.

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

In ASP.NET Core 3,0 e versioni successive i valori predefiniti di navigava sullostesso sito sono stati modificati per evitare conflitti con impostazioni predefinite client incoerenti. Le API seguenti hanno modificato il valore predefinito `SameSiteMode.Lax ` da `-1` a per evitare la creazione di un attributo navigava sullostesso sito per questi cookie:

* <xref:Microsoft.AspNetCore.Http.CookieOptions>usato con [HttpContext. Response. cookies. Append](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*)
* <xref:Microsoft.AspNetCore.Http.CookieBuilder>usato come factory per`CookieOptions`
* [CookiePolicyOptions.MinimumSameSitePolicy](xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy)

::: moniker-end

## <a name="history-and-changes"></a>Cronologia e modifiche

Il supporto di navigava sullostesso sito è stato implementato per la prima volta nel ASP.NET Core 2,0 usando lo [standard 2016 Draft](https://tools.ietf.org/html/draft-west-first-party-cookies-07#section-4.1). Lo standard 2016 è stato acconsentito esplicitamente. Per impostazione predefinita, ASP.NET Core si `Lax` è scelto di impostare diversi cookie. Dopo avere riscontrato diversi [problemi](https://github.com/aspnet/Announcements/issues/318) di autenticazione, la maggior parte dell'utilizzo di navigava sullostesso sito è stata [disabilitata](https://github.com/aspnet/Announcements/issues/348).

Le [patch](https://devblogs.microsoft.com/dotnet/net-core-November-2019/) sono state rilasciate nel 2019 novembre per l'aggiornamento dallo standard 2016 allo standard 2019. [Bozza 2019 della specifica navigava sullostesso sito](https://github.com/aspnet/Announcements/issues/390):

* **Non** è compatibile con le versioni precedenti della bozza 2016. Per ulteriori informazioni, vedere [supporto dei browser meno recenti](#sob) in questo documento.
* Specifica che i cookie vengono `SameSite=Lax` considerati come per impostazione predefinita.
* Specifica i cookie che asseriscono `SameSite=None` in modo esplicito per consentire il recapito tra siti `Secure`deve essere contrassegnato come. `None`è una nuova voce da rifiutare esplicitamente.
* È supportato dalle patch rilasciate per ASP.NET Core 2,1, 2,2 e 3,0. ASP.NET Core 3,1 dispone di supporto navigava sullostesso sito aggiuntivo.
* Viene pianificata per essere abilitata per impostazione predefinita da [Chrome](https://chromestatus.com/feature/5088147346030592) in [feb 2020](https://blog.chromium.org/2019/10/developers-get-ready-for-new.html). Il passaggio a questo standard nei browser è stato avviato in 2019.

## <a name="apis-impacted-by-the-change-from-the-2016-samesite-draft-standard-to-the-2019-draft-standard"></a>API interessate dal passaggio dallo standard Draft 2016 navigava sullostesso sito allo standard 2019 Draft

* [Http. SameSiteMode](xref:Microsoft.AspNetCore.Http.SameSiteMode)
* [CookieOptions. navigava sullostesso sito](xref:Microsoft.AspNetCore.Http.CookieOptions.SameSite)
* [CookieBuilder. navigava sullostesso sito](xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite)
* [CookiePolicyOptions.MinimumSameSitePolicy](xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy)
* <xref:Microsoft.Net.Http.Headers.SameSiteMode?displayProperty=fullName>
* <xref:Microsoft.Net.Http.Headers.SetCookieHeaderValue.SameSite?displayProperty=fullName>

<a name="sob"></a>

## <a name="supporting-older-browsers"></a>Supporto di browser meno recenti

Lo standard 2016 navigava sullostesso sito ha richiesto che i valori sconosciuti debbano essere `SameSite=Strict` considerati valori. Le app a cui è stato eseguito l'accesso da browser meno recenti che supportano lo standard navigava sullostesso sito 2016 possono interrompersi `None`quando ottengono una proprietà navigava sullostesso sito con un valore. Le app Web devono implementare il rilevamento del browser se intendono supportare browser meno recenti. ASP.NET Core non implementa il rilevamento del browser perché i valori degli agenti utente sono altamente volatili e cambiano di frequente. Un punto di estensione <xref:Microsoft.AspNetCore.CookiePolicy> in consente di collegare la logica specifica dell'agente utente.

In `Startup.Configure`aggiungere il codice che chiama <xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy*> prima di <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> chiamare o *qualsiasi* metodo che scrive cookie:

[!code-csharp[](samesite/sample/Startup.cs?name=snippet5&highlight=18-19)]

In `Startup.ConfigureServices`aggiungere codice simile al seguente:

::: moniker range="= aspnetcore-3.1"

[!code-csharp[](samesite/sample/Startup31.cs?name=snippet)]

::: moniker-end

::: moniker range="< aspnetcore-3.1"

[!code-csharp[](samesite/sample/Startup.cs?name=snippet)]

::: moniker-end

Nell'esempio precedente, `MyUserAgentDetectionLib.DisallowsSameSiteNone` è una libreria fornita dall'utente che rileva se l'agente utente non supporta navigava sullostesso sito `None`:

[!code-csharp[](samesite/sample/Startup31.cs?name=snippet2)]

Il codice seguente illustra un metodo `DisallowsSameSiteNone` di esempio:

> [!WARNING]
> Il codice seguente è solo a scopo dimostrativo:
> * Non deve essere considerato completo.
> * Non è gestita né supportata.

[!code-csharp[](samesite/sample/Startup31.cs?name=snippetX)]

## <a name="test-apps-for-samesite-problems"></a>Testare le app per i problemi di navigava sullostesso sito

Le app che interagiscono con i siti remoti, ad esempio tramite l'accesso di terze parti, devono:

* Testare l'interazione su più browser.
* Applicare il [rilevamento e la mitigazione del browser CookiePolicy](#sob) descritti in questo documento.

Testare le app Web usando una versione client che può acconsentire esplicitamente al nuovo comportamento del navigava sullostesso sito. Chrome, Firefox e Chromium Edge hanno tutti nuovi flag di funzionalità di consenso esplicito che possono essere usati per i test. Quando l'app applica le patch di navigava sullostesso sito, testarla con versioni client precedenti, in particolare Safari. Per ulteriori informazioni, vedere [supporto dei browser meno recenti](#sob) in questo documento.

### <a name="test-with-chrome"></a>Eseguire test con Chrome

Chrome 78 + fornisce risultati fuorvianti perché prevede una mitigazione temporanea. La mitigazione temporanea di Chrome 78 + consente ai cookie meno di due minuti di età. Chrome 76 o 77 con i flag di test appropriati abilitati fornisce risultati più accurati. Per testare il nuovo comportamento `chrome://flags/#same-site-by-default-cookies` di navigava sullostesso sito, impostare su **Enabled**. Le versioni precedenti di Chrome (75 e versioni precedenti) vengono segnalate per avere `None` esito negativo con la nuova impostazione. Vedere [supporto di browser meno recenti](#sob) in questo documento.

Google non rende disponibili versioni precedenti di Chrome. Seguire le istruzioni riportate in [scaricare Chromium](https://www.chromium.org/getting-involved/download-chromium) per testare le versioni precedenti di Chrome. Non **scaricare Chrome** dai collegamenti forniti cercando le versioni precedenti di Chrome.

* [Cromo 76 Win64](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Win_x64/664998/)
* [Cromo 74 Win64](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Win_x64/638880/)

A partire dalla versione `80.0.3975.0`Canary, è possibile disabilitare la mitigazione temporanea + post temporanea a scopo di test usando il `--enable-features=SameSiteDefaultChecksMethodRigorously` nuovo flag per consentire il test di siti e servizi nello stato finale finale della funzionalità in cui è stata rimossa la mitigazione. Per ulteriori informazioni, vedere la pagina relativa agli [aggiornamenti di navigava sullostesso sito](https://www.chromium.org/updates/same-site) per progetti Chromium

### <a name="test-with-safari"></a>Eseguire test con Safari

Safari 12 ha implementato rigorosamente la bozza precedente e ha esito negativo quando il nuovo `None` valore si trova in un cookie. `None`viene evitato tramite il codice di rilevamento del browser che [supporta i browser meno recenti](#sob) in questo documento. Testare gli accessi basati su stile sistema operativo Safari 12, Safari 13 e WebKit usando MSAL, ADAL o qualsiasi libreria in uso. Il problema dipende dalla versione del sistema operativo sottostante. I problemi di compatibilità con il nuovo comportamento navigava sullostesso sito sono noti per OSX Mojave (10,14) e iOS 12. L'aggiornamento del sistema operativo a OSX Catalina (10,15) o iOS 13 corregge il problema. Safari attualmente non dispone di un flag di consenso esplicito per il test del nuovo comportamento delle specifiche.

### <a name="test-with-firefox"></a>Eseguire test con Firefox

Il supporto di Firefox per il nuovo standard può essere testato nella versione 68 + scegliendo `about:config` nella pagina con il flag `network.cookie.sameSite.laxByDefault`funzionalità. Non sono stati segnalati problemi di compatibilità con le versioni precedenti di Firefox.

### <a name="test-with-edge-browser"></a>Eseguire test con il browser Microsoft Edge

Edge supporta lo standard navigava sullostesso sito precedente. La versione perimetrale 44 non presenta problemi di compatibilità noti con il nuovo standard.

### <a name="test-with-edge-chromium"></a>Test con Edge (cromo)

I `edge://flags/#same-site-by-default-cookies` flag navigava sullostesso sito sono impostati nella pagina. Nessun problema di compatibilità rilevato con cromo perimetrale.

### <a name="test-with-electron"></a>Test con elettrone

Le versioni di Electron includono versioni precedenti di cromo. Ad esempio, la versione di Electron usata dai team è Chromium 66, che presenta il comportamento precedente. È necessario eseguire test di compatibilità personalizzati con la versione di Electron utilizzata dal prodotto. Vedere [supporto dei browser precedenti](#sob) nella sezione seguente.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Blog di Chromium: sviluppatori: prepararsi per la nuova navigava sullostesso sito = None; Impostazioni sicure del cookie](https://blog.chromium.org/2019/10/developers-get-ready-for-new.html)
* [Spiegazione dei cookie navigava sullostesso sito](https://web.dev/samesite-cookies-explained/)
* [Patch di novembre 2019](https://devblogs.microsoft.com/dotnet/net-core-November-2019/)

 ::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

| Esempio               | Document |
| ----------------- | ------------ |
| [MVC .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC)  | <xref:security/samesite/mvc21> |
| [Razor Pages .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21RazorPages)  | <xref:security/samesite/rp21> |

::: moniker-end

 ::: moniker range=">= aspnetcore-3.0"

| Esempio               | Document |
| ----------------- | ------------ |
| [Pagine di Razor .NET Core](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore31RazorPages)  | <xref:security/samesite/rp31> |

::: moniker-end
