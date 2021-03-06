---
title: Considerazioni sulla sicurezza in gRPC per ASP.NET Core
author: jamesnk
description: Informazioni sulle considerazioni relative alla sicurezza per gRPC per ASP.NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.custom: mvc
ms.date: 07/07/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: grpc/security
ms.openlocfilehash: 8bbe198087f8ba80abfe6b518f8223c719801a85
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 05/04/2020
ms.locfileid: "82774955"
---
# <a name="security-considerations-in-grpc-for-aspnet-core"></a>Considerazioni sulla sicurezza in gRPC per ASP.NET Core

Di [James Newton-King](https://twitter.com/jamesnk)

Questo articolo fornisce informazioni sulla protezione di gRPC con .NET Core.

## <a name="transport-security"></a>Sicurezza del trasporto

i messaggi gRPC vengono inviati e ricevuti tramite HTTP/2. È consigliabile:

* [Transport Layer Security (TLS)](https://tools.ietf.org/html/rfc5246) viene usato per proteggere i messaggi nelle app gRPC di produzione.
* i servizi gRPC devono restare in ascolto e rispondere solo su porte protette.

TLS è configurato in gheppio. Per altre informazioni sulla configurazione degli endpoint di gheppio, vedere la pagina relativa alla [configurazione dell'endpoint gheppio](xref:fundamentals/servers/kestrel#endpoint-configuration).

## <a name="exceptions"></a>Eccezioni

I messaggi di eccezione vengono in genere considerati dati sensibili che non devono essere rivelati a un client. Per impostazione predefinita, gRPC non invia al client i dettagli di un'eccezione generata da un servizio gRPC. Il client riceve invece un messaggio generico che indica che si è verificato un errore. Il recapito dei messaggi di eccezione al client può essere sottoposto a override (ad esempio, in fase di sviluppo o test) con [EnableDetailedErrors](xref:grpc/configuration#configure-services-options). I messaggi di eccezione non devono essere esposti al client nelle app di produzione.

## <a name="message-size-limits"></a>Limiti delle dimensioni dei messaggi

I messaggi in ingresso per i client e i servizi di gRPC vengono caricati in memoria. I limiti delle dimensioni dei messaggi sono un meccanismo che consente di impedire a gRPC di consumare risorse eccessive.

gRPC utilizza i limiti delle dimensioni per messaggio per gestire i messaggi in ingresso e in uscita. Per impostazione predefinita, gRPC limita i messaggi in ingresso a 4 MB. Non esiste alcun limite per i messaggi in uscita.

Sul server, è possibile configurare i limiti dei messaggi gRPC per tutti i servizi in un' `AddGrpc`app con:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc(options =>
    {
        options.MaxReceiveMessageSize = 1 * 1024 * 1024; // 1 MB
        options.MaxSendMessageSize = 1 * 1024 * 1024; // 1 MB
    });
}
```

È anche possibile configurare i limiti per un singolo servizio `AddServiceOptions<TService>`usando. Per ulteriori informazioni sulla configurazione dei limiti delle dimensioni dei messaggi, vedere [configurazione di gRPC](xref:grpc/configuration).

## <a name="client-certificate-validation"></a>Convalida del certificato client

I [certificati client](https://tools.ietf.org/html/rfc5246#section-7.4.4) vengono inizialmente convalidati quando viene stabilita la connessione. Per impostazione predefinita, gheppio non esegue la convalida aggiuntiva del certificato client di una connessione.

È consigliabile che i servizi gRPC protetti dai certificati client usino il pacchetto [Microsoft. AspNetCore. Authentication. Certificate](xref:security/authentication/certauth) . ASP.NET Core Autenticazione della certificazione eseguirà una convalida aggiuntiva per un certificato client, tra cui:

* Il certificato ha un utilizzo chiave esteso valido (EKU)
* Rientra nel periodo di validità
* Controllare la revoca dei certificati
