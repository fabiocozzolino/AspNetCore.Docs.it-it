---
title: Eseguire l'hosting di ASP.NET Core nei contenitori Docker
author: rick-anderson
description: Collegamenti alle risorse che illustrano come eseguire l'hosting di app ASP.NET Core nei contenitori Docker.
ms.author: riande
ms.custom: mvc
ms.date: 01/08/2018
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: host-and-deploy/docker/index
ms.openlocfilehash: 94996f279929f6e3837efb613c99927edbd9143e
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 05/04/2020
ms.locfileid: "82774587"
---
# <a name="host-aspnet-core-in-docker-containers"></a>Eseguire l'hosting di ASP.NET Core nei contenitori Docker

Per apprendere come eseguire l'hosting di app ASP.NET Core in Docker, leggere gli articoli seguenti:

[Introduzione a contenitori e Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/index)  
Descrive la containerizzazione, un approccio allo sviluppo del software in cui un'applicazione o un servizio, le relative dipendenze e la configurazione corrispondente sono inclusi in uno stesso pacchetto come immagine del contenitore. L'immagine può essere testata e quindi distribuita in un host.

[Che cos'è Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/docker-defined)  
Informazioni su Docker, un progetto open source per automatizzare la distribuzione di app come contenitori portabili e autosufficienti eseguibili nel cloud o in locale.

[Terminologia di Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/docker-terminology)  
Informazioni su termini e definizioni relativi alla tecnologia Docker.

[Contenitori, immagini e registri Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/docker-containers-images-registries)  
Informazioni sulle modalità con cui le immagini del contenitore Docker vengono archiviate in un registro immagini per una distribuzione uniforme nei diversi ambienti.

<xref:host-and-deploy/docker/building-net-docker-images> Informazioni sulle procedure per compilare un'app di ASP.NET Core e inserirla in un contenitore Docker. Si analizzano le immagini Docker gestite da Microsoft e si esaminano casi d'uso.

[Strumenti contenitore di Visual Studio](xref:host-and-deploy/docker/visual-studio-tools-for-docker)  
Informazioni su come Visual Studio supporta la compilazione, il debug e l'esecuzione di app ASP.NET Core destinate a .NET Framework o .NET Core in Docker per Windows. Sono supportati sia contenitori Windows che contenitori Linux.

[Eseguire la pubblicazione in Registro Azure Container](/azure/vs-azure-tools-docker-hosting-web-apps-in-docker)  
Informazioni su come usare l'estensione degli Strumenti contenitore di Visual Studio per distribuire un'app ASP.NET Core in un host Docker in Azure con PowerShell.

[Configurare ASP.NET Core per l'utilizzo di server proxy e servizi di bilanciamento del carico](xref:host-and-deploy/proxy-load-balancer)  
Potrebbero essere necessari interventi di configurazione aggiuntivi per le app ospitate dietro a server proxy e a servizi di bilanciamento del carico. Il passaggio delle richieste attraverso un proxy spesso oscura le informazioni sulla richiesta originale, ad esempio lo schema e l'indirizzo IP del client. Potrebbe essere necessario inoltrare alcune informazioni sulla richiesta manualmente all'app.

[GC con contenitori Docker e Small](xref:performance/memory#sc) Viene descritta la selezione GC con piccoli contenitori.