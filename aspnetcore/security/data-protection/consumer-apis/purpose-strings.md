---
title: Stringhe di scopo in ASP.NET Core
author: rick-anderson
description: Informazioni sul modo in cui vengono usate le stringhe di scopo in ASP.NET Core le API di protezione dei dati.
ms.author: riande
ms.date: 10/14/2016
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/data-protection/consumer-apis/purpose-strings
ms.openlocfilehash: 9cdc762a6dd3cbf6e248d0b72d915d09dee25eb0
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 05/04/2020
ms.locfileid: "82774171"
---
# <a name="purpose-strings-in-aspnet-core"></a>Stringhe di scopo in ASP.NET Core

<a name="data-protection-consumer-apis-purposes"></a>

I `CreateProtector` componenti che `IDataProtectionProvider` utilizzano devono passare un parametro per *finalità* univoche al metodo. Il *parametro* degli scopi è intrinseco alla sicurezza del sistema di protezione dei dati, in quanto fornisce l'isolamento tra i consumer di crittografia, anche se le chiavi di crittografia radice sono le stesse.

Quando un consumer specifica uno scopo, la stringa per finalità viene utilizzata insieme alle chiavi di crittografia radice per derivare le sottochiavi crittografiche univoche per tale consumer. Questo consente di isolare il consumer da tutti gli altri consumer crittografici nell'applicazione: nessun altro componente può leggere i payload e non è in grado di leggere i payload di altri componenti. Questo isolamento esegue anche il rendering di intere categorie non realizzabili di attacchi contro il componente.

![Esempio di diagramma scopo](purpose-strings/_static/purposes.png)

Nel diagramma precedente, `IDataProtector` le istanze a e B **non** sono in grado di leggere i payload degli altri, ma solo dei relativi.

Non è necessario che la stringa di scopo sia segreta. Dovrebbe essere semplicemente univoco nel senso che nessun altro componente ben funzionante fornirà la stessa stringa di scopo.

>[!TIP]
> L'uso dello spazio dei nomi e del nome del tipo del componente che utilizza le API di protezione dati è una regola empirica, come in pratica queste informazioni non sono in conflitto.
>
>Un componente creato da Contoso che è responsabile per la creazione di token di portanti può utilizzare contoso. Security. BearerToken come stringa di scopo. O, anche meglio, potrebbe usare contoso. Security. BearerToken. v1 come stringa di scopo. L'aggiunta del numero di versione consente a una versione futura di usare contoso. Security. BearerToken. v2 come scopo e le diverse versioni sarebbero completamente isolate l'una dall'altra per quanto riguarda i payload.

Dato che il parametro purposes a è una matrice di `CreateProtector` stringhe, il valore precedente potrebbe essere `[ "Contoso.Security.BearerToken", "v1" ]`stato invece specificato come. Questo consente di stabilire una gerarchia di scopi e apre la possibilità di scenari di multi-tenant con il sistema di protezione dei dati.

<a name="data-protection-contoso-purpose"></a>

>[!WARNING]
> I componenti non devono consentire l'input dell'utente non attendibile come unica fonte di input per la catena di scopi.
>
>Si consideri, ad esempio, un componente contoso. Messaging. SecureMessage responsabile dell'archiviazione dei messaggi protetti. Se il componente di messaggistica protetta venisse chiamato `CreateProtector([ username ])`, un utente malintenzionato potrebbe creare un account con il nome utente "contoso. Security. BearerToken" nel tentativo di ottenere il componente `CreateProtector([ "Contoso.Security.BearerToken" ])`da chiamare, causando inavvertitamente il sistema di messaggistica sicura per la mentazione dei payload che potrebbero essere percepiti come token di autenticazione.
>
>Una catena di scopi migliore per il componente di messaggistica `CreateProtector([ "Contoso.Messaging.SecureMessage", "User: username" ])`sarebbe, che fornisce un isolamento appropriato.

L'isolamento fornito da e i comportamenti degli `IDataProtectionProvider`scopi `IDataProtector`, e sono i seguenti:

* Per un determinato `IDataProtectionProvider` oggetto, il `CreateProtector` metodo creerà un `IDataProtector` oggetto in modo univoco associato all' `IDataProtectionProvider` oggetto che lo ha creato e al parametro purposes passato nel metodo.

* Il parametro purpose non può essere null. Se gli scopi sono specificati come matrice, significa che la matrice non deve essere di lunghezza zero e che tutti gli elementi della matrice non devono essere null. Uno scopo di stringa vuoto è tecnicamente consentito, ma è sconsigliato.

* Due argomenti per finalità sono equivalenti se e solo se contengono le stesse stringhe (usando un operatore di confronto ordinale) nello stesso ordine. Un argomento a scopo singolo equivale alla matrice a scopo di singolo elemento corrispondente.

* Due `IDataProtector` oggetti sono equivalenti se e solo se vengono creati da oggetti `IDataProtectionProvider` equivalenti con parametri di scopi equivalenti.

* Per un determinato `IDataProtector` oggetto, una chiamata a `Unprotect(protectedData)` restituirà l'originale `unprotectedData` se e solo se `protectedData := Protect(unprotectedData)` per un oggetto `IDataProtector` equivalente.

> [!NOTE]
> Il caso in cui un componente sceglie intenzionalmente una stringa di scopo nota come conflitto con un altro componente non viene considerato. Questo componente è essenzialmente considerato dannoso e questo sistema non è progettato per fornire garanzie di sicurezza nel caso in cui il codice dannoso sia già in esecuzione all'interno del processo di lavoro.
