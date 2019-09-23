---
title: Ospitare e distribuire ASP.NET Core Server Blazer
author: guardrex
description: Informazioni su come ospitare e distribuire un'app Server Blazer usando ASP.NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/07/2019
uid: host-and-deploy/blazor/server
ms.openlocfilehash: a393d620924d847e674a09972515a8130a15fc6a
ms.sourcegitcommit: 092061c4f6ef46ed2165fa84de6273d3786fb97e
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/13/2019
ms.locfileid: "70964282"
---
# <a name="host-and-deploy-blazor-server"></a><span data-ttu-id="90831-103">Ospitare e distribuire il server Blazer</span><span class="sxs-lookup"><span data-stu-id="90831-103">Host and deploy Blazor Server</span></span>

<span data-ttu-id="90831-104">Di [Luke Latham](https://github.com/guardrex), [Rainer Stropek](https://www.timecockpit.com) e [Daniel Roth](https://github.com/danroth27)</span><span class="sxs-lookup"><span data-stu-id="90831-104">By [Luke Latham](https://github.com/guardrex), [Rainer Stropek](https://www.timecockpit.com), and [Daniel Roth](https://github.com/danroth27)</span></span>

## <a name="host-configuration-values"></a><span data-ttu-id="90831-105">Valori di configurazione dell'host</span><span class="sxs-lookup"><span data-stu-id="90831-105">Host configuration values</span></span>

<span data-ttu-id="90831-106">Le [app del server Blazer](xref:blazor/hosting-models#blazor-server) possono accettare [valori di configurazione host generici](xref:fundamentals/host/generic-host#host-configuration).</span><span class="sxs-lookup"><span data-stu-id="90831-106">[Blazor Server apps](xref:blazor/hosting-models#blazor-server) can accept [Generic Host configuration values](xref:fundamentals/host/generic-host#host-configuration).</span></span>

## <a name="deployment"></a><span data-ttu-id="90831-107">Distribuzione</span><span class="sxs-lookup"><span data-stu-id="90831-107">Deployment</span></span>

<span data-ttu-id="90831-108">Con il [modello di hosting del server Blazer](xref:blazor/hosting-models#blazor-server), Blazer viene eseguito sul server dall'interno di un'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="90831-108">Using the [Blazor Server hosting model](xref:blazor/hosting-models#blazor-server), Blazor is executed on the server from within an ASP.NET Core app.</span></span> <span data-ttu-id="90831-109">Gli aggiornamenti dell'interfaccia utente, la gestione degli eventi e le chiamate JavaScript vengono gestite tramite una connessione [SignalR](xref:signalr/introduction).</span><span class="sxs-lookup"><span data-stu-id="90831-109">UI updates, event handling, and JavaScript calls are handled over a [SignalR](xref:signalr/introduction) connection.</span></span>

<span data-ttu-id="90831-110">È necessario un server Web in grado di ospitare un'app ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="90831-110">A web server capable of hosting an ASP.NET Core app is required.</span></span> <span data-ttu-id="90831-111">Visual Studio include il modello di progetto **App server Blazor** (modello `blazorserverside` quando si usa il comando [dotnet new](/dotnet/core/tools/dotnet-new)).</span><span class="sxs-lookup"><span data-stu-id="90831-111">Visual Studio includes the **Blazor Server App** project template (`blazorserverside` template when using the [dotnet new](/dotnet/core/tools/dotnet-new) command).</span></span>

## <a name="scalability"></a><span data-ttu-id="90831-112">Scalabilità</span><span class="sxs-lookup"><span data-stu-id="90831-112">Scalability</span></span>

<span data-ttu-id="90831-113">Pianificare una distribuzione per sfruttare al meglio l'infrastruttura disponibile per un'app del server blazer.</span><span class="sxs-lookup"><span data-stu-id="90831-113">Plan a deployment to make the best use of the available infrastructure for a Blazor Server app.</span></span> <span data-ttu-id="90831-114">Vedere le risorse seguenti per risolvere la scalabilità delle app del server Blazer:</span><span class="sxs-lookup"><span data-stu-id="90831-114">See the following resources to address Blazor Server app scalability:</span></span>

* [<span data-ttu-id="90831-115">Nozioni fondamentali sulle app del server Blazer</span><span class="sxs-lookup"><span data-stu-id="90831-115">Fundamentals of Blazor Server apps</span></span>](xref:blazor/hosting-models#blazor-server)
* <xref:security/blazor/server>

### <a name="deployment-server"></a><span data-ttu-id="90831-116">Server di distribuzione</span><span class="sxs-lookup"><span data-stu-id="90831-116">Deployment server</span></span>

<span data-ttu-id="90831-117">Quando si considera la scalabilità di un singolo server (scalabilità verticale), la memoria disponibile per un'app è probabilmente la prima risorsa che verrà esaurita dall'app quando le richieste degli utenti aumentano.</span><span class="sxs-lookup"><span data-stu-id="90831-117">When considering the scalability of a single server (scale up), the memory available to an app is likely the first resource that the app will exhaust as user demands increase.</span></span> <span data-ttu-id="90831-118">La memoria disponibile sul server influiscono su:</span><span class="sxs-lookup"><span data-stu-id="90831-118">The available memory on the server affects the:</span></span>

* <span data-ttu-id="90831-119">Numero di circuiti attivi che un server è in grado di supportare.</span><span class="sxs-lookup"><span data-stu-id="90831-119">Number of active circuits that a server can support.</span></span>
* <span data-ttu-id="90831-120">Latenza dell'interfaccia utente nel client.</span><span class="sxs-lookup"><span data-stu-id="90831-120">UI latency on the client.</span></span>

<span data-ttu-id="90831-121">Per istruzioni sulla creazione di app Server Blazer sicure e scalabili <xref:security/blazor/server>, vedere.</span><span class="sxs-lookup"><span data-stu-id="90831-121">For guidance on building secure and scalable Blazor server apps, see <xref:security/blazor/server>.</span></span>

<span data-ttu-id="90831-122">Ogni circuito utilizza circa 250 KB di memoria per un'app di tipo *Hello World*minima.</span><span class="sxs-lookup"><span data-stu-id="90831-122">Each circuit uses approximately 250 KB of memory for a minimal *Hello World*-style app.</span></span> <span data-ttu-id="90831-123">La dimensione di un circuito dipende dal codice dell'app e dai requisiti di manutenzione dello stato associati a ogni componente.</span><span class="sxs-lookup"><span data-stu-id="90831-123">The size of a circuit depends on the app's code and the state maintenance requirements associated with each component.</span></span> <span data-ttu-id="90831-124">Si consiglia di misurare le richieste di risorse durante lo sviluppo dell'applicazione e dell'infrastruttura, ma la linea di base seguente può essere un punto di partenza per la pianificazione della destinazione di distribuzione: Se si prevede che l'app supporti 5.000 utenti simultanei, valutare la possibilità di stabilire un budget di almeno 1,3 GB di memoria del server per l'app (o ~ 273 KB per utente).</span><span class="sxs-lookup"><span data-stu-id="90831-124">We recommend that you measure resource demands during development for your app and infrastructure, but the following baseline can be a starting point in planning your deployment target: If you expect your app to support 5,000 concurrent users, consider budgeting at least 1.3 GB of server memory to the app (or ~273 KB per user).</span></span>

### <a name="signalr-configuration"></a><span data-ttu-id="90831-125">Configurazione di SignalR</span><span class="sxs-lookup"><span data-stu-id="90831-125">SignalR configuration</span></span>

<span data-ttu-id="90831-126">Le app del server Blazer usano ASP.NET Core SignalR per comunicare con il browser.</span><span class="sxs-lookup"><span data-stu-id="90831-126">Blazor Server apps use ASP.NET Core SignalR to communicate with the browser.</span></span> <span data-ttu-id="90831-127">[Le condizioni di hosting e scalabilità di SignalR](xref:signalr/publish-to-azure-web-app) si applicano alle app del server blazer.</span><span class="sxs-lookup"><span data-stu-id="90831-127">[SignalR's hosting and scaling conditions](xref:signalr/publish-to-azure-web-app) apply to Blazor Server apps.</span></span>

<span data-ttu-id="90831-128">Blazer funziona meglio quando si usano WebSocket come trasporto SignalR grazie a latenza, affidabilità e [sicurezza](xref:signalr/security)inferiori.</span><span class="sxs-lookup"><span data-stu-id="90831-128">Blazor works best when using WebSockets as the SignalR transport due to lower latency, reliability, and [security](xref:signalr/security).</span></span> <span data-ttu-id="90831-129">Il polling prolungato viene usato da SignalR quando WebSocket non è disponibile o quando l'app è configurata in modo esplicito per l'uso del polling prolungato.</span><span class="sxs-lookup"><span data-stu-id="90831-129">Long Polling is used by SignalR when WebSockets isn't available or when the app is explicitly configured to use Long Polling.</span></span> <span data-ttu-id="90831-130">Quando si esegue la distribuzione nel servizio app Azure, configurare l'app per l'uso di WebSocket nelle impostazioni portale di Azure per il servizio.</span><span class="sxs-lookup"><span data-stu-id="90831-130">When deploying to Azure App Service, configure the app to use WebSockets in the Azure portal settings for the service.</span></span> <span data-ttu-id="90831-131">Per informazioni dettagliate sulla configurazione dell'app per il servizio app Azure, vedere le [linee guida sulla pubblicazione di SignalR](xref:signalr/publish-to-azure-web-app).</span><span class="sxs-lookup"><span data-stu-id="90831-131">For details on configuring the app for Azure App Service, see the [SignalR publishing guidelines](xref:signalr/publish-to-azure-web-app).</span></span>

<span data-ttu-id="90831-132">È consigliabile usare il [servizio Azure SignalR](/azure/azure-signalr) per le app del server blazer.</span><span class="sxs-lookup"><span data-stu-id="90831-132">We recommend using the [Azure SignalR Service](/azure/azure-signalr) for Blazor Server apps.</span></span> <span data-ttu-id="90831-133">Il servizio consente la scalabilità verticale di un'app del server blazer a un numero elevato di connessioni di SignalR simultanee.</span><span class="sxs-lookup"><span data-stu-id="90831-133">The service allows for scaling up a Blazor Server app to a large number of concurrent SignalR connections.</span></span> <span data-ttu-id="90831-134">Inoltre, la portata globale del servizio SignalR e i Data Center a prestazioni elevate consentono di ridurre significativamente la latenza a causa della geografia.</span><span class="sxs-lookup"><span data-stu-id="90831-134">In addition, the SignalR service's global reach and high-performance data centers significantly aid in reducing latency due to geography.</span></span>

### <a name="measure-network-latency"></a><span data-ttu-id="90831-135">Misurare la latenza di rete</span><span class="sxs-lookup"><span data-stu-id="90831-135">Measure network latency</span></span>

<span data-ttu-id="90831-136">L' [interoperabilità JS](xref:blazor/javascript-interop) può essere usata per misurare la latenza di rete, come illustrato nell'esempio seguente:</span><span class="sxs-lookup"><span data-stu-id="90831-136">[JS interop](xref:blazor/javascript-interop) can be used to measure network latency, as the following example demonstrates:</span></span>

```cshtml
@inject IJSRuntime JS

@if (latency is null)
{
    <span>Calculating...</span>
}
else
{
    <span>@(latency.Value.TotalMilliseconds)ms</span>
}

@code
{
    private DateTime startTime;
    private TimeSpan? latency;

    protected override async Task OnInitializedAsync()
    {
        startTime = DateTime.UtcNow;
        var _ = await JS.InvokeAsync<string>("toString");
        latency = DateTime.UtcNow - startTime;
    }
}
```

<span data-ttu-id="90831-137">Per un'esperienza dell'interfaccia utente ragionevole, è consigliabile una latenza di interfaccia utente prolungata di 250ms o meno.</span><span class="sxs-lookup"><span data-stu-id="90831-137">For a reasonable UI experience, we recommend a sustained UI latency of 250ms or less.</span></span>