---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

---
# <a name="configuration-in-aspnet-core"></a>Configurazione in ASP.NET Core

Di [Rick Anderson](https://twitter.com/RickAndMSFT) e [Kirk Larkin](https://twitter.com/serpent5)

::: moniker range=">= aspnetcore-3.0"

La configurazione in ASP.NET Core viene eseguita utilizzando uno o più [provider di configurazione](#cp). I provider di configurazione leggono i dati di configurazione da coppie chiave-valore usando un'ampia gamma di origini di configurazione:

* File di impostazioni, ad esempio *appSettings. JSON*
* Variabili di ambiente
* Insieme di credenziali chiave di Azure
* Configurazione app di Azure
* Argomenti della riga di comando
* Provider personalizzati, installati o creati
* File della directory
* Oggetti .NET in memoria

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

<a name="default"></a>

## <a name="default-configuration"></a>Configurazione predefinita

ASP.NET Core app Web create con [DotNet New](/dotnet/core/tools/dotnet-new) o Visual Studio generano il codice seguente:

[!code-csharp[](index/samples/3.x/ConfigSample/Program.cs?name=snippet&highlight=9)]

 <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*> fornisce la configurazione predefinita per l'app nell'ordine seguente:

1. [ChainedConfigurationProvider](xref:Microsoft.Extensions.Configuration.ChainedConfigurationSource) : aggiunge un oggetto esistente `IConfiguration` come origine. Nel caso di configurazione predefinita, aggiunge la configurazione [host](#hvac) e la imposta come prima origine per la configurazione dell' _app_ .
1. [appSettings. JSON](#appsettingsjson) con il [provider di configurazione JSON](#file-configuration-provider).
1. *appSettings.* `Environment` *. JSON* con il [provider di configurazione JSON](#file-configuration-provider). Ad esempio, *appSettings*. ***Produzione***. *JSON* e *appSettings*. ***Sviluppo***. *JSON*.
1. [Segreti dell'app](xref:security/app-secrets) quando l'app viene eseguita nell' `Development` ambiente.
1. Variabili di ambiente che usano il [provider di configurazione delle variabili di ambiente](#evcp).
1. Argomenti della riga di comando che usano il [provider di configurazione della riga di comando](#command-line).

I provider di configurazione aggiunti successivamente sostituiscono le impostazioni di chiave precedenti. Se, ad esempio, `MyKey` è impostato sia in *appSettings. JSON* che nell'ambiente, viene utilizzato il valore dell'ambiente. Utilizzando i provider di configurazione predefiniti, il [provider di configurazione della riga di comando](#command-line-configuration-provider) esegue l'override di tutti gli altri provider.

Per ulteriori informazioni su `CreateDefaultBuilder` , vedere [impostazioni predefinite del generatore](xref:fundamentals/host/generic-host#default-builder-settings).

Il codice seguente Visualizza i provider di configurazione abilitati nell'ordine in cui sono stati aggiunti:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Index2.cshtml.cs?name=snippet)]

### <a name="appsettingsjson"></a>appsettings.json

Si consideri il file *appSettings. JSON* seguente:

[!code-json[](index/samples/3.x/ConfigSample/appsettings.json)]

Il codice seguente del [download dell'esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) Visualizza diverse impostazioni di configurazione precedenti:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

Il valore predefinito <xref:Microsoft.Extensions.Configuration.Json.JsonConfigurationProvider> carica la configurazione nell'ordine seguente:

1. *appsettings.json*
1. *appSettings.* `Environment` *. JSON* : ad esempio, *appSettings*. ***Produzione***. *JSON* e *appSettings*. ***Sviluppo***. file *JSON* . La versione dell'ambiente del file viene caricata in base a [IHostingEnvironment. EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.EnvironmentName*). Per altre informazioni, vedere <xref:fundamentals/environments>.

*appSettings*. `Environment` . i valori *JSON* eseguono l'override delle chiavi in *appSettings. JSON*. Ad esempio, per impostazione predefinita:

* In fase di sviluppo, *appSettings*. ***Sviluppo***. la configurazione *JSON* sovrascrive i valori trovati in *appSettings. JSON*.
* In produzione, *appSettings*. ***Produzione***. la configurazione *JSON* sovrascrive i valori trovati in *appSettings. JSON*. Ad esempio, quando si distribuisce l'app in Azure.

<a name="optpat"></a>

### <a name="bind-hierarchical-configuration-data-using-the-options-pattern"></a>Associare i dati di configurazione gerarchici usando il modello di opzioni

[!INCLUDE[](~/includes/bind.md)]

Utilizzando la configurazione [predefinita](#default) , *appSettings. JSON* e *appSettings.* `Environment` i file con *estensione JSON* sono abilitati con [reloadOnChange: true](https://github.com/dotnet/extensions/blob/release/3.1/src/Hosting/Hosting/src/Host.cs#L74-L75). Modifiche apportate a *appSettings. JSON* e *appSettings.* `Environment` il file con *estensione JSON* ***dopo*** l'avvio dell'app viene letto dal [provider di configurazione JSON](#jcp).

Per informazioni sull'aggiunta di altri file di configurazione JSON, vedere [provider di configurazione JSON](#jcp) in questo documento.

<a name="security"></a>

## <a name="security-and-secret-manager"></a>Gestione della sicurezza e del segreto

Linee guida sui dati di configurazione:

* Non archiviare mai la password o altri dati sensibili nel codice del provider di configurazione o in file di configurazione di testo normale. Il [gestore del segreto](xref:security/app-secrets) può essere usato per archiviare i segreti in fase di sviluppo.
* Non usare i segreti di produzione in ambienti di sviluppo o di test.
* Specificare i segreti all'esterno del progetto in modo che non possano essere inavvertitamente inviati a un repository del codice sorgente.

Per [impostazione predefinita](#default), [Secret Manager](xref:security/app-secrets) legge le impostazioni di configurazione dopo *appSettings. JSON* e *appSettings.* `Environment` *. JSON*.

Per ulteriori informazioni sull'archiviazione di password o altri dati sensibili:

* <xref:fundamentals/environments>
* <xref:security/app-secrets>: Include consigli sull'uso delle variabili di ambiente per archiviare dati riservati. Il gestore dei segreti USA il [provider di configurazione file](#fcp) per archiviare i segreti utente in un file JSON nel sistema locale.

[Azure Key Vault](https://azure.microsoft.com/services/key-vault/) archivia in modo sicuro i segreti delle app ASP.NET Core. Per altre informazioni, vedere <xref:security/key-vault-configuration>.

<a name="evcp"></a>

## <a name="environment-variables"></a>Variabili di ambiente

Usando la configurazione [predefinita](#default) , <xref:Microsoft.Extensions.Configuration.EnvironmentVariables.EnvironmentVariablesConfigurationProvider> carica la configurazione dalle coppie chiave-valore della variabile di ambiente dopo aver letto *appSettings. JSON*, *appSettings.* `Environment` *. JSON*e [gestione segreta](xref:security/app-secrets). Pertanto, i valori di chiave letti dall'ambiente eseguono l'override dei valori letti da *appSettings. JSON*, *appSettings.* `Environment` *. JSON*e gestione segreta.

[!INCLUDE[](~/includes/environmentVarableColon.md)]

I `set` comandi seguenti:

* Impostare le chiavi e i valori di ambiente dell' [esempio precedente](#appsettingsjson) in Windows.
* Testare le impostazioni quando si usa il [download di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample). Il `dotnet run` comando deve essere eseguito nella directory del progetto.

```dotnetcli
set MyKey="My key from Environment"
set Position__Title=Environment_Editor
set Position__Name=Environment_Rick
dotnet run
```

Le impostazioni di ambiente precedenti:

* Vengono impostati solo nei processi avviati dalla finestra di comando in cui sono stati impostati.
* Non verrà letto dai browser avviati con Visual Studio.

Per impostare le chiavi e i valori di ambiente in Windows, è possibile usare i comandi [Setx](/windows-server/administration/windows-commands/setx) seguenti. Diversamente da `set` , `setx` le impostazioni sono rese permanente. `/M`imposta la variabile nell'ambiente di sistema. Se l' `/M` opzione non viene usata, viene impostata una variabile di ambiente utente.

```cmd
setx MyKey "My key from setx Environment" /M
setx Position__Title Setx_Environment_Editor /M
setx Position__Name Environment_Rick /M
```

Per verificare che i comandi precedenti eseguano l'override di *appSettings. JSON* e *appSettings.* `Environment` *. JSON*:

* Con Visual Studio: chiudere e riavviare Visual Studio.
* Con l'interfaccia della riga di comando: avviare una nuova finestra di comando e immettere `dotnet run` .

Chiamare <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> con una stringa per specificare un prefisso per le variabili di ambiente:

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Program.cs?name=snippet4&highlight=12)]

Nel codice precedente:

* `config.AddEnvironmentVariables(prefix: "MyCustomPrefix_")`viene aggiunto dopo i [provider di configurazione predefiniti](#default). Per un esempio di ordinamento dei provider di configurazione, vedere [provider di configurazione JSON](#jcp).
* Le variabili di ambiente impostate con il `MyCustomPrefix_` prefisso sostituiscono i [provider di configurazione predefiniti](#default). Sono incluse le variabili di ambiente senza il prefisso.

Il prefisso viene rimosso quando vengono lette le coppie chiave-valore di configurazione.

I comandi seguenti verificano il prefisso personalizzato:

```dotnetcli
set MyCustomPrefix_MyKey="My key with MyCustomPrefix_ Environment"
set MyCustomPrefix_Position__Title=Editor_with_customPrefix
set MyCustomPrefix_Position__Name=Environment_Rick_cp
dotnet run
```

La [configurazione predefinita](#default) carica le variabili di ambiente e gli argomenti della riga di comando preceduti da `DOTNET_` e `ASPNETCORE_` . I `DOTNET_` `ASPNETCORE_` prefissi e vengono usati da ASP.NET Core per la [configurazione dell'host e dell'app](xref:fundamentals/host/generic-host#host-configuration), ma non per la configurazione dell'utente. Per ulteriori informazioni sulla configurazione dell'host e dell'app, vedere [.NET Generic Host](xref:fundamentals/host/generic-host).

In [app Azure servizio](https://azure.microsoft.com/services/app-service/)selezionare **impostazione nuova applicazione** nella pagina **Impostazioni > configurazione** . App Azure impostazioni dell'applicazione del servizio sono:

* Crittografati a riposo e trasmessi su un canale crittografato.
* Esposto come variabile di ambiente.

Per altre informazioni, vedere [App di Azure: Eseguire l'override della configurazione delle app usando il portale di Azure](xref:host-and-deploy/azure-apps/index#override-app-configuration-using-the-azure-portal).

Per informazioni sulle stringhe di connessione del database di Azure, vedere [prefissi della stringa di connessione](#constr) .

<a name="clcp"></a>

## <a name="command-line"></a>Riga di comando

Utilizzando la configurazione [predefinita](#default) , <xref:Microsoft.Extensions.Configuration.CommandLine.CommandLineConfigurationProvider> carica la configurazione dalle coppie chiave-valore dell'argomento della riga di comando dopo le origini di configurazione seguenti:

* *appSettings. JSON* e *appSettings*. `Environment` . file *JSON* .
* [Segreti dell'app (gestione segreto)](xref:security/app-secrets) nell'ambiente di sviluppo.
* Variabili di ambiente.

Per [impostazione predefinita](#default), i valori di configurazione impostati nei valori di configurazione di override della riga di comando impostati con tutti gli altri provider di configurazione.

### <a name="command-line-arguments"></a>Argomenti della riga di comando

Il comando seguente imposta le chiavi e i valori usando `=` :

```dotnetcli
dotnet run MyKey="My key from command line" Position:Title=Cmd Position:Name=Cmd_Rick
```

Il comando seguente imposta le chiavi e i valori usando `/` :

```dotnetcli
dotnet run /MyKey "Using /" /Position:Title=Cmd_ /Position:Name=Cmd_Rick
```

Il comando seguente imposta le chiavi e i valori usando `--` :

```dotnetcli
dotnet run --MyKey "Using --" --Position:Title=Cmd-- --Position:Name=Cmd--Rick
```

Valore chiave:

* Deve seguire `=` oppure la chiave deve avere un prefisso `--` o `/` quando il valore segue uno spazio.
* Non è obbligatorio se `=` si usa. Ad esempio: `MySetting=`.

All'interno dello stesso comando, non combinare coppie chiave-valore dell'argomento della riga di comando che usano `=` con coppie chiave-valore che usano uno spazio.

### <a name="switch-mappings"></a>Mapping di sostituzione

I mapping switch consentono la logica di sostituzione del nome **chiave** . Fornire un dizionario di sostituzioni switch al <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> metodo.

Quando viene utilizzato il dizionario dei mapping di sostituzione, nel dizionario viene controllata la presenza di una chiave corrispondente alla chiave fornita da un argomento della riga di comando. Se la chiave della riga di comando viene trovata nel dizionario, il valore del dizionario viene passato di nuovo per impostare la coppia chiave-valore nella configurazione dell'app. Un mapping di sostituzione è necessario per le chiavi della riga di comando con un trattino singolo (`-`) come prefisso.

Regole principali del dizionario dei mapping di sostituzione:

* Le opzioni devono iniziare con `-` o `--` .
* Il dizionario dei mapping di sostituzione non deve contenere chiavi duplicate.

Per usare un dizionario dei mapping switch, passarlo alla chiamata a `AddCommandLine` :

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramSwitch.cs?name=snippet&highlight=10-18,23)]

Il codice seguente mostra i valori chiave per le chiavi sostituite:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test3.cshtml.cs?name=snippet)]

Eseguire il comando seguente per testare la sostituzione della chiave:

```dotnetcli
dotnet run -k1=value1 -k2 value2 --alt3=value2 /alt4=value3 --alt5 value5 /alt6 value6
```

Nota: attualmente `=` non è possibile usare per impostare i valori di sostituzione delle chiavi con un trattino singolo `-` . Vedere [il problema in GitHub](https://github.com/dotnet/extensions/issues/3059).

Il comando seguente funziona per testare la sostituzione della chiave:

```dotnetcli
dotnet run -k1 value1 -k2 value2 --alt3=value2 /alt4=value3 --alt5 value5 /alt6 value6
```

Per le app che usano i mapping di sostituzione, la chiamata a `CreateDefaultBuilder` non deve passare argomenti. La `CreateDefaultBuilder` chiamata al metodo `AddCommandLine` non include opzioni mappate e non è possibile passare il dizionario di mapping switch a `CreateDefaultBuilder` . La soluzione non passa gli argomenti a `CreateDefaultBuilder` ma per consentire al metodo del `ConfigurationBuilder` metodo `AddCommandLine` di elaborare sia gli argomenti sia il dizionario di mapping delle opzioni.

## <a name="hierarchical-configuration-data"></a>Dati di configurazione gerarchici

L'API di configurazione legge i dati di configurazione gerarchici rendendo flat i dati gerarchici con l'uso di un delimitatore nelle chiavi di configurazione.

Il [download di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) contiene il file *appSettings. JSON* seguente:

[!code-json[](index/samples/3.x/ConfigSample/appsettings.json)]

Il codice seguente del [download dell'esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) Visualizza diverse impostazioni di configurazione:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

Il modo migliore per leggere i dati di configurazione gerarchici consiste nell'usare il modello di opzioni. Per altre informazioni, vedere [associare dati di configurazione gerarchici](#optpat) in questo documento.

I metodi <xref:Microsoft.Extensions.Configuration.ConfigurationSection.GetSection*> e <xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*> sono disponibili per isolare le sezioni e gli elementi figlio di una sezione nei dati di configurazione. Questi metodi sono descritti più avanti in [GetSection, GetChildren ed Exists](#getsection).

<!--
[Azure Key Vault configuration provider](xref:security/key-vault-configuration) implement change detection.
-->

## <a name="configuration-keys-and-values"></a>Chiavi e valori di configurazione

Chiavi di configurazione:

* Non fanno distinzione tra maiuscole e minuscole. Ad esempio, `ConnectionString` e `connectionstring` vengono considerate chiavi equivalenti.
* Se una chiave e un valore vengono impostati in più provider di configurazione, viene utilizzato il valore dell'ultimo provider aggiunto. Per ulteriori informazioni, vedere [configurazione predefinita](#default).
* Chiavi gerarchiche
  * Nell'ambito dell'API di configurazione, il separatore due punti (`:`) funziona in tutte le piattaforme.
  * Nelle variabili di ambiente, un separatore due punti potrebbe non funzionare in tutte le piattaforme. Un doppio carattere di sottolineatura, `__` , è supportato da tutte le piattaforme e viene convertito automaticamente in due punti `:` .
  * In Azure Key Vault, le chiavi gerarchiche utilizzano `--` come separatore. Il [provider di configurazione Azure Key Vault](xref:security/key-vault-configuration) sostituisce automaticamente `--` con un `:` quando i segreti vengono caricati nella configurazione dell'app.
* Il <xref:Microsoft.Extensions.Configuration.ConfigurationBinder> supporta l'associazione di matrici agli oggetti usando gli indici delle matrici nelle chiavi di configurazione. L'associazione di matrici è descritta nella sezione [Associare una matrice a una classe](#boa).

Valori di configurazione:

* Sono stringhe.
* I valori null non possono essere archiviati nella configurazione o associati a oggetti.

<a name="cp"></a>

## <a name="configuration-providers"></a>Provider di configurazione

La tabella seguente mostra i provider di configurazione disponibili per le app ASP.NET Core.

| Provider | Fornisce la configurazione da |
| ---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

---- | Titolo---: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

------------------ | | [Provider di configurazione Azure Key Vault](xref:security/key-vault-configuration) | Azure Key Vault | | [Provider di configurazione app Azure](/azure/azure-app-configuration/quickstart-aspnet-core-app) | Configurazione app Azure | | [Provider di configurazione della riga di comando](#clcp) | Parametri della riga di comando | | [Provider di configurazione personalizzato](#custom-configuration-provider) | Origine personalizzata | | [Provider di configurazione delle variabili di ambiente](#evcp) | Variabili di ambiente | | [Provider di configurazione file](#file-configuration-provider) | File INI, JSON e XML | | [Provider di configurazione chiave per file](#key-per-file-configuration-provider) | File di directory | | [Provider di configurazione della memoria](#memory-configuration-provider) | Raccolte in memoria | | [Gestore Secret](xref:security/app-secrets) | File nella directory del profilo utente |

Le origini di configurazione vengono lette nell'ordine in cui sono specificati i provider di configurazione. Ordinare i provider di configurazione nel codice in base alle priorità per le origini di configurazione sottostanti richieste dall'app.

Una sequenza tipica di provider di configurazione è:

1. *appsettings.json*
1. *appSettings*. `Environment` . *JSON*
1. [Gestione segreta](xref:security/app-secrets)
1. Variabili di ambiente che usano il [provider di configurazione delle variabili di ambiente](#evcp).
1. Argomenti della riga di comando che usano il [provider di configurazione della riga di comando](#command-line-configuration-provider).

Una procedura comune consiste nell'aggiungere il provider di configurazione della riga di comando per ultimo in una serie di provider per consentire agli argomenti della riga di comando di eseguire l'override del set di configurazione da parte degli altri provider.

La sequenza di provider precedente viene utilizzata nella [configurazione predefinita](#default).

<a name="constr"></a>

### <a name="connection-string-prefixes"></a>Prefissi della stringa di connessione

L'API di configurazione dispone di regole di elaborazione speciali per quattro variabili di ambiente della stringa di connessione. Queste stringhe di connessione sono necessarie per configurare le stringhe di connessione di Azure per l'ambiente dell'app. Le variabili di ambiente con i prefissi visualizzati nella tabella vengono caricate nell'app con la [configurazione predefinita](#default) o quando non viene fornito alcun prefisso a `AddEnvironmentVariables` .

| Prefisso della stringa di connessione | Provider |
| ---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

------------ | Titolo---: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

---- | | `CUSTOMCONNSTR_` | Provider personalizzato | | `MYSQLCONNSTR_` | [MySQL](https://www.mysql.com/) |
 MySQL | `SQLAZURECONNSTR_` | SQL Server del [database SQL di Azure](https://azure.microsoft.com/services/sql-database/) |
 | `SQLCONNSTR_` | [SQL Server](https://www.microsoft.com/sql-server/)|

Quando una variabile di ambiente viene individuata e caricata nella configurazione con uno qualsiasi dei quattro prefissi indicati nella tabella:

* La chiave di configurazione viene creata rimuovendo il prefisso della variabile di ambiente e aggiungendo una sezione per la chiave di configurazione (`ConnectionStrings`).
* Viene creata una nuova coppia chiave-valore della configurazione che rappresenta il provider di connessione al database (ad eccezione di `CUSTOMCONNSTR_`, che non ha un provider dichiarato).

| Chiave della variabile di ambiente | Chiave di configurazione convertita | Voce di configurazione del provider                                                    |
| ---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

------------ | Titolo---: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-------------- | Titolo---: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

---------------------------------------- | | `CUSTOMCONNSTR_{KEY} `   | `ConnectionStrings:{KEY}`   | Voce di configurazione non creata.                                                | | `MYSQLCONNSTR_{KEY}`     | `ConnectionStrings:{KEY}`   | Chiave: `ConnectionStrings:{KEY}_ProviderName` :<br>Valore: `MySql.Data.MySqlClient` | | `SQLAZURECONNSTR_{KEY}`   |  `ConnectionStrings:{KEY}`   | Chiave: `ConnectionStrings:{KEY}_ProviderName` :<br>Valore: `System.Data.SqlClient` | | `SQLCONNSTR_{KEY}`        |  `ConnectionStrings:{KEY}`   | Chiave: `ConnectionStrings:{KEY}_ProviderName` :<br>Valore`System.Data.SqlClient`  |

<a name="jcp"></a>

### <a name="json-configuration-provider"></a>Provider di configurazione JSON

<xref:Microsoft.Extensions.Configuration.Json.JsonConfigurationProvider>Carica la configurazione dalle coppie chiave-valore del file JSON.

Gli overload possono specificare:

* Se il file è facoltativo.
* Se la configurazione viene ricaricata se viene modificato il file.

Esaminare il codice seguente:

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSON.cs?name=snippet&highlight=12-14)]

Il codice precedente:

* Configura il provider di configurazione JSON per caricare il file *config. JSON* con le opzioni seguenti:
  * `optional: true`: Il file è facoltativo.
  * `reloadOnChange: true`: Il file viene ricaricato quando vengono salvate le modifiche.
* Legge i [provider di configurazione predefiniti](#default) prima del file *config. JSON* . Impostazioni nell'impostazione di sostituzione del file *config. JSON* nei provider di configurazione predefiniti, tra cui il [provider di configurazione delle variabili di ambiente](#evcp) e il provider di configurazione della riga di [comando](#clcp).

In genere ***non*** si vuole che un file JSON personalizzato esegua l'override dei valori impostati nel [provider di configurazione delle variabili di ambiente](#evcp) e nel provider di configurazione della riga di [comando](#clcp).

Il codice seguente cancella tutti i provider di configurazione e aggiunge diversi provider di configurazione:

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSON2.cs?name=snippet)]

Nel codice precedente, Settings in *config. JSON* e *config*. `Environment` . file *JSON* :

* Eseguire l'override delle impostazioni nel file *appSettings. JSON* e *appSettings*. `Environment` . file *JSON* .
* Viene sottoposto a override dalle impostazioni del [provider di configurazione delle variabili di ambiente](#evcp) e del provider di configurazione della riga di [comando](#clcp).

Il [download di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) contiene il file *config. JSON* seguente:

[!code-json[](index/samples/3.x/ConfigSample/MyConfig.json)]

Il codice seguente del [download dell'esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) Visualizza diverse impostazioni di configurazione precedenti:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

<a name="fcp"></a>

## <a name="file-configuration-provider"></a>Provider di configurazione file

<xref:Microsoft.Extensions.Configuration.FileConfigurationProvider> è la classe base per il caricamento della configurazione dal file system. I provider di configurazione seguenti derivano da `FileConfigurationProvider` :

* [Provider di configurazione INI](#ini-configuration-provider)
* [Provider di configurazione JSON](#jcp)
* [Provider di configurazione XML](#xml-configuration-provider)

### <a name="ini-configuration-provider"></a>Provider di configurazione INI

<xref:Microsoft.Extensions.Configuration.Ini.IniConfigurationProvider> carica la configurazione da coppie chiave-valore di file INI in fase di esecuzione.

Il codice seguente cancella tutti i provider di configurazione e aggiunge diversi provider di configurazione:

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramINI.cs?name=snippet&highlight=10-30)]

Nel codice precedente, le impostazioni in *MyIniConfig. ini* e *MyIniConfig*. `Environment` . i file *ini* vengono sottoposti a override dalle impostazioni nel:

* [Provider di configurazione delle variabili di ambiente](#evcp)
* [Provider di configurazione della riga di comando](#clcp).

Il [download di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) contiene il seguente file *MyIniConfig. ini* :

[!code-ini[](index/samples/3.x/ConfigSample/MyIniConfig.ini)]

Il codice seguente del [download dell'esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) Visualizza diverse impostazioni di configurazione precedenti:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

### <a name="xml-configuration-provider"></a>Provider di configurazione XML

Il <xref:Microsoft.Extensions.Configuration.Xml.XmlConfigurationProvider> carica la configurazione da coppie chiave-valore di file XML in fase di esecuzione.

Il codice seguente cancella tutti i provider di configurazione e aggiunge diversi provider di configurazione:

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramXML.cs?name=snippet)]

Nel codice precedente, le impostazioni in *MyXMLFile. XML* e *MyXMLFile*. `Environment` . i file *XML* vengono sottoposti a override dalle impostazioni in:

* [Provider di configurazione delle variabili di ambiente](#evcp)
* [Provider di configurazione della riga di comando](#clcp).

Il [download di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) contiene il seguente file *MyXMLFile. XML* :

[!code-xml[](index/samples/3.x/ConfigSample/MyXMLFile.xml)]

Il codice seguente del [download dell'esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) Visualizza diverse impostazioni di configurazione precedenti:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

La ripetizione di elementi che usano lo stesso nome di elemento funziona se si usa l'attributo `name` per distinguere gli elementi:

[!code-xml[](index/samples/3.x/ConfigSample/MyXMLFile3.xml)]

Il codice seguente legge il file di configurazione precedente e visualizza le chiavi e i valori:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/XML/Index.cshtml.cs?name=snippet)]

È possibile usare attributi per fornire valori:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <key attribute="value" />
  <section>
    <key attribute="value" />
  </section>
</configuration>
```

Il file di configurazione precedente carica le chiavi seguenti con `value`:

* key:attribute
* section:key:attribute

## <a name="key-per-file-configuration-provider"></a>Provider di configurazione chiave per file

Il <xref:Microsoft.Extensions.Configuration.KeyPerFile.KeyPerFileConfigurationProvider> usa i file di una directory come coppie chiave-valore della configurazione. La chiave è il nome del file. Il valore contiene il contenuto del file. Il provider di configurazione chiave per file viene usato negli scenari di hosting di Docker.

Per attivare la configurazione chiave-per-file, chiamare il metodo di estensione <xref:Microsoft.Extensions.Configuration.KeyPerFileConfigurationBuilderExtensions.AddKeyPerFile*> su un'istanza di <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder>. Il `directoryPath` per i file deve essere un percorso assoluto.

Gli overload consentono di specificare:

* Un delegato `Action<KeyPerFileConfigurationSource>` che configura l'origine.
* Se la directory è facoltativa e il percorso della directory.

Il doppio carattere di sottolineatura (`__`) viene usato come delimitatore per le chiavi di configurazione nei nomi dei file. Ad esempio, il nome di file `Logging__LogLevel__System` produce la chiave di configurazione `Logging:LogLevel:System`.

Chiamare `ConfigureAppConfiguration` quando si crea l'host per specificare la configurazione dell'app:

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    var path = Path.Combine(
        Directory.GetCurrentDirectory(), "path/to/files");
    config.AddKeyPerFile(directoryPath: path, optional: true);
})
```

<a name="mcp"></a>

## <a name="memory-configuration-provider"></a>Provider di configurazione della memoria

Il <xref:Microsoft.Extensions.Configuration.Memory.MemoryConfigurationProvider> usa una raccolta in memoria come coppie chiave-valore della configurazione.

Il codice seguente aggiunge una raccolta di memoria al sistema di configurazione:

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramArray.cs?name=snippet6)]

Il codice seguente del [download dell'esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample) Visualizza le impostazioni di configurazione precedenti:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

Nel codice precedente, `config.AddInMemoryCollection(Dict)` viene aggiunto dopo i [provider di configurazione predefiniti](#default). Per un esempio di ordinamento dei provider di configurazione, vedere [provider di configurazione JSON](#jcp).

Vedere [associare una matrice](#boa) per un altro esempio di utilizzo di `MemoryConfigurationProvider` .

## <a name="getvalue"></a>GetValue

[`ConfigurationBinder.GetValue<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.GetValue*)estrae un singolo valore dalla configurazione con una chiave specificata e lo converte nel tipo specificato:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestNum.cshtml.cs?name=snippet)]

Nel codice precedente, se `NumberKey` non è stato trovato nella configurazione, viene usato il valore predefinito di `99` .

## <a name="getsection-getchildren-and-exists"></a>GetSection, GetChildren ed Exists

Per gli esempi che seguono, prendere in considerazione il seguente file *MySubsection. JSON* :

[!code-json[](index/samples/3.x/ConfigSample/MySubsection.json)]

Il codice seguente aggiunge *MySubsection. JSON* ai provider di configurazione:

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSONsection.cs?name=snippet)]

### <a name="getsection"></a>GetSection

[IConfiguration. GetSection](xref:Microsoft.Extensions.Configuration.IConfiguration.GetSection*) restituisce una sottosezione di configurazione con la chiave della sottosezione specificata.

Il codice seguente restituisce i valori per `section1` :

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestSection.cshtml.cs?name=snippet)]

Il codice seguente restituisce i valori per `section2:subsection0` :

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestSection2.cshtml.cs?name=snippet)]

`GetSection` non restituisce mai `null`. Se non viene trovata una sezione corrispondente, viene restituita una `IConfigurationSection` vuota.

Quando `GetSection` restituisce una sezione corrispondente, <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Value> non viene compilata. Quando la sezione esiste, vengono restituiti <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Key> e <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Path>.

### <a name="getchildren-and-exists"></a>GetChildren ed EXISTS

Il codice seguente chiama [IConfiguration. GetChildren](xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*) e restituisce i valori per `section2:subsection0` :

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestSection4.cshtml.cs?name=snippet)]

Il codice precedente chiama [ConfigurationExtensions. Exists](xref:Microsoft.Extensions.Configuration.ConfigurationExtensions.Exists*) per verificare che la sezione esista:

 <a name="boa"></a>

## <a name="bind-an-array"></a>Associare una matrice

[ConfigurationBinder. Bind](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*) supporta l'associazione di matrici a oggetti usando gli indici di matrice nelle chiavi di configurazione. Qualsiasi formato di matrice che espone un segmento di chiave numerica è in grado di associare array a una matrice di classi [poco](https://wikipedia.org/wiki/Plain_Old_CLR_Object) .

Si consideri il file con *estensione JSON* dal [download di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample):

[!code-json[](index/samples/3.x/ConfigSample/MyArray.json)]

Il codice seguente aggiunge il file *Array. JSON* ai provider di configurazione:

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSONarray.cs?name=snippet)]

Il codice seguente legge la configurazione e Visualizza i valori:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Array.cshtml.cs?name=snippet)]

Il codice precedente restituisce l'output seguente:

```text
Index: 0  Value: value00
Index: 1  Value: value10
Index: 2  Value: value20
Index: 3  Value: value40
Index: 4  Value: value50
```

Nell'output precedente, index 3 ha un valore `value40` , corrispondente a `"4": "value40",` in *ArrayList. JSON*. Gli indici di matrice associati sono continui e non sono associati all'indice della chiave di configurazione. Il binder di configurazione non è in grado di associare valori null o di creare voci null negli oggetti associati

Il codice seguente carica la `array:entries` configurazione con il <xref:Microsoft.Extensions.Configuration.MemoryConfigurationBuilderExtensions.AddInMemoryCollection*> metodo di estensione:

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramArray.cs?name=snippet)]

Il codice seguente legge la configurazione in `arrayDict` `Dictionary` e Visualizza i valori:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Array.cshtml.cs?name=snippet)]

Il codice precedente restituisce l'output seguente:

```text
Index: 0  Value: value0
Index: 1  Value: value1
Index: 2  Value: value2
Index: 3  Value: value4
Index: 4  Value: value5
```

L'indice &num;3 nell'oggetto associato contiene i dati di configurazione per la chiave di configurazione `array:4` e il relativo valore `value4`. Quando i dati di configurazione contenenti una matrice sono associati, gli indici di matrice nelle chiavi di configurazione vengono usati per scorrere i dati di configurazione durante la creazione dell'oggetto. Un valore null non può essere mantenuto nei dati di configurazione e una voce con valore null non viene creata in un oggetto associato quando una matrice nelle chiavi di configurazione ignora uno o più indici.

È possibile specificare l'elemento di configurazione mancante per &num; l'indice 3 prima di eseguire il binding all' `ArrayExample` istanza da qualsiasi provider di configurazione che legga la &num; coppia chiave/valore dell'indice 3. Si consideri il seguente file *valore3. JSON* dal Download di esempio:

[!code-json[](index/samples/3.x/ConfigSample/Value3.json)]

Il codice seguente include la configurazione per *valore3. JSON* e `arrayDict` `Dictionary` :

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramArray.cs?name=snippet2)]

Il codice seguente legge la configurazione precedente e Visualizza i valori:

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Array.cshtml.cs?name=snippet)]

Il codice precedente restituisce l'output seguente:

```text
Index: 0  Value: value0
Index: 1  Value: value1
Index: 2  Value: value2
Index: 3  Value: value3
Index: 4  Value: value4
Index: 5  Value: value5
```

I provider di configurazione personalizzati non devono implementare l'associazione di matrici.

## <a name="custom-configuration-provider"></a>Provider di configurazione personalizzato

L'app di esempio dimostra come creare un provider di configurazione di base che legge le coppie chiave-valore di configurazione da un database usando [Entity Framework (EF)](/ef/core/).

Il provider ha le caratteristiche seguenti:

* Il database in memoria di Entity Framework viene usato a scopo dimostrativo. Per usare un database che richiede una stringa di connessione, implementare un `ConfigurationBuilder` secondario per fornire la stringa di connessione da un altro provider di configurazione.
* Il provider legge una tabella di database in una configurazione all'avvio. Il provider non esegue una query sul database per ogni chiave.
* Il ricaricamento in caso di modifica non è implementato, quindi l'aggiornamento del database dopo l'avvio dell'app non ha alcun effetto sulla configurazione dell'app.

Definire un'entità `EFConfigurationValue` per l'archiviazione dei valori di configurazione nel database.

*Models/EFConfigurationValue.cs*:

[!code-csharp[](index/samples/3.x/ConfigurationSample/Models/EFConfigurationValue.cs?name=snippet1)]

Aggiungere `EFConfigurationContext` per archiviare i valori configurati e accedervi.

*EFConfigurationProvider/EFConfigurationContext.cs*:

[!code-csharp[](index/samples/3.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationContext.cs?name=snippet1)]

Creare una classe che implementi <xref:Microsoft.Extensions.Configuration.IConfigurationSource>.

*EFConfigurationProvider/EFConfigurationSource.cs*:

[!code-csharp[](index/samples/3.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationSource.cs?name=snippet1)]

Creare il provider di configurazione personalizzato ereditando da <xref:Microsoft.Extensions.Configuration.ConfigurationProvider>. Il provider di configurazione inizializza il database quando è vuoto. Poiché le [chiavi di configurazione non fanno distinzione tra maiuscole](#keys)e minuscole, il dizionario utilizzato per inizializzare il database viene creato con l'operatore di confronto senza distinzione tra maiuscole e minuscole ([StringComparer. OrdinalIgnoreCase](xref:System.StringComparer.OrdinalIgnoreCase)).

*EFConfigurationProvider/EFConfigurationProvider.cs*:

[!code-csharp[](index/samples/3.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationProvider.cs?name=snippet1)]

Un metodo di estensione `AddEFConfiguration` consente di aggiungere l'origine di configurazione a un `ConfigurationBuilder`.

*Extensions/EntityFrameworkExtensions.cs*:

[!code-csharp[](index/samples/3.x/ConfigurationSample/Extensions/EntityFrameworkExtensions.cs?name=snippet1)]

L'esempio di codice seguente mostra come usare il `EFConfigurationProvider` personalizzato in *Program.cs*:

[!code-csharp[](index/samples/3.x/ConfigurationSample/Program.cs?name=snippet_Program&highlight=29-30)]

<a name="acs"></a>

## <a name="access-configuration-in-startup"></a>Accedi alla configurazione all'avvio

Il codice seguente consente di visualizzare i dati di configurazione nei `Startup` metodi:

[!code-csharp[](index/samples/3.x/ConfigSample/StartupKey.cs?name=snippet&highlight=13,18)]

Per un esempio di accesso alla configurazione usando metodi di servizio di avvio, vedere [Avvio dell'applicazione: Metodi pratici](xref:fundamentals/startup#convenience-methods).

## <a name="access-configuration-in-razor-pages"></a>Accedi alla configurazione nelle Razor pagine

Il codice seguente consente di visualizzare i dati di configurazione in una Razor pagina:

[!code-cshtml[](index/samples/3.x/ConfigSample/Pages/Test5.cshtml)]

Nel codice seguente, `MyOptions` viene aggiunto al contenitore del servizio con <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure*> e associato alla configurazione:

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Startup3.cs?name=snippet_Example2)]

Il markup seguente utilizza la [`@inject`](xref:mvc/views/razor#inject) Razor direttiva per risolvere e visualizzare i valori delle opzioni:

[!code-cshtml[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Pages/Test3.cshtml)]

## <a name="access-configuration-in-a-mvc-view-file"></a>Accedere alla configurazione in un file di visualizzazione MVC

Il codice seguente consente di visualizzare i dati di configurazione in una visualizzazione MVC:

[!code-cshtml[](index/samples/3.x/ConfigSample/Views/Home2/Index.cshtml)]

## <a name="configure-options-with-a-delegate"></a>Configurare le opzioni con un delegato

Le opzioni configurate in un delegato eseguono l'override dei valori impostati nei provider di configurazione.

La configurazione delle opzioni con un delegato è illustrata nell'esempio 2 nell'app di esempio.

Nel codice seguente <xref:Microsoft.Extensions.Options.IConfigureOptions%601> viene aggiunto un servizio al contenitore del servizio. Usa un delegato per configurare i valori per `MyOptions` :

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Startup2.cs?name=snippet_Example2)]

Il codice seguente consente di visualizzare i valori delle opzioni:

[!code-csharp[](options/samples/3.x/OptionsSample/Pages/Test2.cshtml.cs?name=snippet)]

Nell'esempio precedente, i valori di `Option1` e `Option2` vengono specificati in *appSettings. JSON* e quindi sottoposti a override dal delegato configurato.

<a name="hvac"></a>

## <a name="host-versus-app-configuration"></a>Host e configurazione delle app

Prima che l'app venga configurata e avviata, viene configurato e avviato un *host*. L'host è responsabile della gestione dell'avvio e della durata delle app. Sia l'app che l'host vengono configurati tramite i provider di configurazione descritti in questo argomento. Anche le coppie chiave-valore di configurazione dell'host sono incluse nella configurazione dell'app. Per altre informazioni su come vengono usati i provider di configurazione quando viene creato l'host e sugli effetti delle origini di configurazione sull'host di configurazione, vedere <xref:fundamentals/index#host>.

<a name="dhc"></a>

## <a name="default-host-configuration"></a>Configurazione host predefinita

Per informazioni dettagliate sulla configurazione predefinita quando viene usato l'[host Web](xref:fundamentals/host/web-host), vedere la [versione di questo argomento per ASP.NET Core 2.2](/aspnet/core/fundamentals/configuration/?view=aspnetcore-2.2).

* La configurazione dell'host viene specificata da:
  * Variabili di ambiente con prefisso `DOTNET_` (ad esempio, `DOTNET_ENVIRONMENT` ) utilizzando il [provider di configurazione delle variabili di ambiente](#environment-variables-configuration-provider). Il prefisso (`DOTNET_`) viene rimosso al caricamento delle coppie chiave-valore della configurazione.
  * Argomenti della riga di comando che usano il [provider di configurazione della riga di comando](#command-line-configuration-provider).
* La configurazione predefinita dell'host Web viene stabilita (`ConfigureWebHostDefaults`) nel modo seguente:
  * Kestrel viene usato come server Web e configurato con i provider di configurazione dell'app.
  * Aggiungere il middleware di filtro host.
  * Aggiungere il middleware delle intestazioni inoltrate se la variabile di ambiente `ASPNETCORE_FORWARDEDHEADERS_ENABLED` è impostata su `true`.
  * Abilitare l'integrazione di IIS.

## <a name="other-configuration"></a>Altra configurazione

Questo argomento riguarda solo la *configurazione dell'app*. Altri aspetti dell'esecuzione e dell'hosting di app ASP.NET Core sono configurati usando i file di configurazione non trattati in questo argomento:

* *Launch. JSON* / *launchSettings. JSON* sono i file di configurazione degli strumenti per l'ambiente di sviluppo, descritti:
  * In <xref:fundamentals/environments#development> .
  * Nella documentazione in cui vengono usati i file per configurare ASP.NET Core app per scenari di sviluppo.
* *Web. config* è un file di configurazione del server, descritto negli argomenti seguenti:
  * <xref:host-and-deploy/iis/index>
  * <xref:host-and-deploy/aspnet-core-module>

Per ulteriori informazioni sulla migrazione della configurazione dell'app da versioni precedenti di ASP.NET, vedere <xref:migration/proper-to-2x/index#store-configurations> .

## <a name="add-configuration-from-an-external-assembly"></a>Aggiungere la configurazione da un assembly esterno

Un'implementazione <xref:Microsoft.AspNetCore.Hosting.IHostingStartup> consente l'aggiunta di miglioramenti a un'app all'avvio, da un assembly esterno alla classe `Startup` dell'app. Per altre informazioni, vedere <xref:fundamentals/configuration/platform-specific-configuration>.

## <a name="additional-resources"></a>Risorse aggiuntive

* [Codice sorgente configurazione](https://github.com/dotnet/extensions/tree/master/src/Configuration)
* <xref:fundamentals/configuration/options>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

La configurazione delle app in ASP.NET Core si basa su coppie chiave-valore stabilite dai *provider di configurazione*. I provider di configurazione leggono i dati di configurazione in coppie chiave-valore da un'ampia gamma di origini di configurazione:

* Insieme di credenziali chiave di Azure
* Configurazione app di Azure
* Argomenti della riga di comando
* Provider personalizzati (installati o creati)
* File della directory
* Variabili di ambiente
* Oggetti .NET in memoria
* File di impostazioni

I pacchetti di configurazione per gli scenari di provider di configurazione comuni ([Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration/)) sono inclusi nel [metapacchetto Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app).

Gli esempi di codice che seguono e quelli inclusi nell'app di esempio usano lo spazio dei nomi <xref:Microsoft.Extensions.Configuration>:

```csharp
using Microsoft.Extensions.Configuration;
```

Il *modello di opzioni* è un'estensione dei concetti di configurazione descritti in questo argomento. Le opzioni usano le classi per rappresentare i gruppi di impostazioni correlate. Per altre informazioni, vedere <xref:fundamentals/configuration/options>.

[Visualizzare o scaricare il codice di esempio](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples) ([procedura per il download](xref:index#how-to-download-a-sample))

## <a name="host-versus-app-configuration"></a>Host e configurazione delle app

Prima che l'app venga configurata e avviata, viene configurato e avviato un *host*. L'host è responsabile della gestione dell'avvio e della durata delle app. Sia l'app che l'host vengono configurati tramite i provider di configurazione descritti in questo argomento. Anche le coppie chiave-valore di configurazione dell'host sono incluse nella configurazione dell'app. Per altre informazioni su come vengono usati i provider di configurazione quando viene creato l'host e sugli effetti delle origini di configurazione sull'host di configurazione, vedere <xref:fundamentals/index#host>.

## <a name="other-configuration"></a>Altra configurazione

Questo argomento riguarda solo la *configurazione dell'app*. Altri aspetti dell'esecuzione e dell'hosting di app ASP.NET Core sono configurati usando i file di configurazione non trattati in questo argomento:

* *Launch. JSON* / *launchSettings. JSON* sono i file di configurazione degli strumenti per l'ambiente di sviluppo, descritti:
  * In <xref:fundamentals/environments#development> .
  * Nella documentazione in cui vengono usati i file per configurare ASP.NET Core app per scenari di sviluppo.
* *Web. config* è un file di configurazione del server, descritto negli argomenti seguenti:
  * <xref:host-and-deploy/iis/index>
  * <xref:host-and-deploy/aspnet-core-module>

Per ulteriori informazioni sulla migrazione della configurazione dell'app da versioni precedenti di ASP.NET, vedere <xref:migration/proper-to-2x/index#store-configurations> .

## <a name="default-configuration"></a>Configurazione predefinita

Le app basate sui modelli [dotnet new](/dotnet/core/tools/dotnet-new) di ASP.NET Core chiamano <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> durante la compilazione di un host. `CreateDefaultBuilder` fornisce la configurazione predefinita per l'app nell'ordine seguente:

La configurazione seguente si applica alle app che usano l'[host Web](xref:fundamentals/host/web-host). Per informazioni dettagliate sulla configurazione predefinita quando viene usato l'[host generico](xref:fundamentals/host/generic-host), vedere la [versione più recente di questo argomento](xref:fundamentals/configuration/index).

* La configurazione dell'host viene specificata da:
  * Variabili di ambiente con prefisso `ASPNETCORE_` (ad esempio `ASPNETCORE_ENVIRONMENT`) che usano il [provider di configurazione delle variabili di ambiente](#environment-variables-configuration-provider). Il prefisso (`ASPNETCORE_`) viene rimosso al caricamento delle coppie chiave-valore della configurazione.
  * Argomenti della riga di comando che usano il [provider di configurazione della riga di comando](#command-line-configuration-provider).
* La configurazione dell'app viene fornita da:
  * *appsettings.json* mediante il [provider di configurazione dei file](#file-configuration-provider).
  * *appsettings.{Ambiente}.json* mediante il [provider di configurazione dei file](#file-configuration-provider).
  * [Strumento di gestione dei segreti](xref:security/app-secrets) quando l'app viene eseguita nell'ambiente `Development` usando l'assembly di ingresso.
  * Variabili di ambiente che usano il [provider di configurazione delle variabili di ambiente](#environment-variables-configuration-provider).
  * Argomenti della riga di comando che usano il [provider di configurazione della riga di comando](#command-line-configuration-provider).

## <a name="security"></a>Sicurezza

Per proteggere i dati di configurazione sensibili, adottare le pratiche seguenti:

* Non archiviare mai la password o altri dati sensibili nel codice del provider di configurazione o in file di configurazione di testo normale.
* Non usare i segreti di produzione in ambienti di sviluppo o di test.
* Specificare i segreti all'esterno del progetto in modo che non possano essere inavvertitamente inviati a un repository del codice sorgente.

Per altre informazioni, vedere gli argomenti seguenti:

* <xref:fundamentals/environments>
* <xref:security/app-secrets>: Include consigli sull'uso delle variabili di ambiente per archiviare dati riservati. Secret Manager usa il provider di configurazione dei file per archiviare i segreti utente in un file JSON nel sistema locale. Il provider di configurazione dei file è descritto più avanti in questo argomento.

[Azure Key Vault](https://azure.microsoft.com/services/key-vault/) archivia in modo sicuro i segreti delle app ASP.NET Core. Per altre informazioni, vedere <xref:security/key-vault-configuration>.

## <a name="hierarchical-configuration-data"></a>Dati di configurazione gerarchici

L'API di configurazione è in grado di mantenere i dati di configurazione gerarchici rendendoli flat grazie all'uso di un delimitatore nelle chiavi di configurazione.

Nel file JSON seguente, esistono quattro chiavi in una gerarchia strutturata di due sezioni:

```json
{
  "section0": {
    "key0": "value",
    "key1": "value"
  },
  "section1": {
    "key0": "value",
    "key1": "value"
  }
}
```

Quando il file viene letto nella configurazione, vengono create chiavi univoche per mantenere la struttura di dati gerarchici originale dell'origine di configurazione. Le sezioni e le chiavi vengono rese flat usando due punti (`:`) per mantenere la struttura originale:

* section0:key0
* section0:key1
* section1:key0
* section1:key1

I metodi <xref:Microsoft.Extensions.Configuration.ConfigurationSection.GetSection*> e <xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*> sono disponibili per isolare le sezioni e gli elementi figlio di una sezione nei dati di configurazione. Questi metodi sono descritti più avanti in [GetSection, GetChildren ed Exists](#getsection-getchildren-and-exists).

## <a name="conventions"></a>Convenzioni

### <a name="sources-and-providers"></a>Origini e provider

All'avvio dell'app, le origini di configurazione vengono lette nell'ordine con cui sono specificati i rispettivi provider di configurazione.

I provider di configurazione che implementano il rilevamento delle modifiche sono in grado di ricaricare la configurazione quando viene modificata un'impostazione sottostante. Ad esempio, il provider di configurazione dei file (descritto più avanti in questo argomento) e il [provider di configurazione di Azure Key Vault](xref:security/key-vault-configuration) implementano il rilevamento delle modifiche.

<xref:Microsoft.Extensions.Configuration.IConfiguration> è disponibile nel contenitore di [inserimento delle dipendenze](xref:fundamentals/dependency-injection) dell'app. <xref:Microsoft.Extensions.Configuration.IConfiguration>può essere inserito in una Razor pagina <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> o MVC <xref:Microsoft.AspNetCore.Mvc.Controller> per ottenere la configurazione per la classe.

Negli esempi seguenti il `_config` campo viene usato per accedere ai valori di configurazione:

```csharp
public class IndexModel : PageModel
{
    private readonly IConfiguration _config;

    public IndexModel(IConfiguration config)
    {
        _config = config;
    }
}
```

```csharp
public class HomeController : Controller
{
    private readonly IConfiguration _config;

    public HomeController(IConfiguration config)
    {
        _config = config;
    }
}
```

I provider di configurazione non possono usare l'inserimento delle dipendenze, perché non è disponibile quando vengono configurati dall'host.

### <a name="keys"></a>Chiavi

Le chiavi di configurazione adottano le convenzioni seguenti:

* Per le chiavi non viene fatta distinzione tra maiuscole e minuscole. Ad esempio, `ConnectionString` e `connectionstring` vengono considerate chiavi equivalenti.
* Se un valore per la stessa chiave viene impostato dallo stesso provider di configurazione o da provider diversi, viene usato l'ultimo valore impostato per la chiave.
* Chiavi gerarchiche
  * Nell'ambito dell'API di configurazione, il separatore due punti (`:`) funziona in tutte le piattaforme.
  * Nelle variabili di ambiente, un separatore due punti potrebbe non funzionare in tutte le piattaforme. Il doppio carattere di sottolineatura (`__`) è supportato da tutte le piattaforme e viene convertito automaticamente nei due punti.
  * In Azure Key Vault, le chiavi gerarchiche usano `--` (due trattini) come separatore. Scrivere il codice per sostituire i trattini con i due punti quando i segreti vengono caricati nella configurazione dell'app.
* Il <xref:Microsoft.Extensions.Configuration.ConfigurationBinder> supporta l'associazione di matrici agli oggetti usando gli indici delle matrici nelle chiavi di configurazione. L'associazione di matrici è descritta nella sezione [Associare una matrice a una classe](#bind-an-array-to-a-class).

### <a name="values"></a>Valori

I valori di configurazione adottano le convenzioni seguenti:

* I valori sono stringhe.
* I valori null non possono essere archiviati nella configurazione o associati a oggetti.

## <a name="providers"></a>Providers

La tabella seguente mostra i provider di configurazione disponibili per le app ASP.NET Core.

| Provider | Fornisce la configurazione da&hellip; |
| ---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

---- | Titolo---: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

------------------ | | [Provider di configurazione Azure Key Vault](xref:security/key-vault-configuration) (argomenti*sulla sicurezza* ) | Azure Key Vault | | [Provider di configurazione app Azure](/azure/azure-app-configuration/quickstart-aspnet-core-app) (documentazione di Azure) | Configurazione app Azure | | [Provider di configurazione della riga di comando](#command-line-configuration-provider) | Parametri della riga di comando | | [Provider di configurazione personalizzato](#custom-configuration-provider) | Origine personalizzata | | [Provider di configurazione delle variabili di ambiente](#environment-variables-configuration-provider) | Variabili di ambiente | | [Provider di configurazione file](#file-configuration-provider) | File (INI, JSON, XML) | | [Provider di configurazione chiave per file](#key-per-file-configuration-provider) | File di directory | | [Provider di configurazione della memoria](#memory-configuration-provider) | Raccolte in memoria | | [Segreti utente (gestione Secret)](xref:security/app-secrets) (argomenti*sulla sicurezza* ) | File nella directory del profilo utente |

Le origini di configurazione vengono lette nell'ordine in cui vengono specificati i rispetti provider di configurazione all'avvio. I provider di configurazione descritti in questo argomento sono descritti in ordine alfabetico, non nell'ordine in cui il codice li dispone. Ordinare i provider di configurazione nel codice in base alle priorità per le origini di configurazione sottostanti richieste dall'app.

Una sequenza tipica di provider di configurazione è:

1. Files (*appSettings. JSON*, *appSettings. { Environment}. JSON*, dove `{Environment}` è l'ambiente host corrente dell'app)
1. [Insieme di credenziali chiave Azure](xref:security/key-vault-configuration)
1. [Segreti utente (Secret Manager)](xref:security/app-secrets) (solo nell'ambiente di sviluppo)
1. Variabili di ambiente
1. Argomenti della riga di comando

È pratica comune posizionare il provider di configurazione della riga di comando per ultimo in una serie di provider per consentire agli argomenti della riga di comando di sostituire la configurazione impostata da altri provider.

La sequenza di provider precedente viene utilizzata quando viene inizializzato un nuovo generatore host con `CreateDefaultBuilder` . Per altre informazioni, vedere la sezione [Configurazione predefinita](#default-configuration).

## <a name="configure-the-host-builder-with-useconfiguration"></a>Configurare il generatore di host con UseConfiguration

Per configurare il generatore di host, chiamare <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseConfiguration*> sul generatore di host con la configurazione.

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args)
{
    var dict = new Dictionary<string, string>
    {
        {"MemoryCollectionKey1", "value1"},
        {"MemoryCollectionKey2", "value2"}
    };

    var config = new ConfigurationBuilder()
        .AddInMemoryCollection(dict)
        .Build();

    return WebHost.CreateDefaultBuilder(args)
        .UseConfiguration(config)
        .UseStartup<Startup>();
}
```

## <a name="configureappconfiguration"></a>ConfigureAppConfiguration

Chiamare `ConfigureAppConfiguration` quando si crea l'host per specificare i provider di configurazione dell'app, oltre a quelli aggiunti automaticamente da `CreateDefaultBuilder`:

[!code-csharp[](index/samples/2.x/ConfigurationSample/Program.cs?name=snippet_Program&highlight=20)]

### <a name="override-previous-configuration-with-command-line-arguments"></a>Sostituire la configurazione precedente con argomenti della riga di comando

Per fornire la configurazione dell'app che può essere sostituita con argomenti della riga di comando, chiamare `AddCommandLine` per ultimo:

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    // Call other providers here
    config.AddCommandLine(args);
})
```

### <a name="remove-providers-added-by-createdefaultbuilder"></a>Rimuovere i provider aggiunti da CreateDefaultBuilder

Per rimuovere i provider aggiunti da `CreateDefaultBuilder` , chiamare prima [Clear](/dotnet/api/system.collections.generic.icollection-1.clear) in [IConfigurationBuilder. Sources](xref:Microsoft.Extensions.Configuration.IConfigurationBuilder.Sources) :

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.Sources.Clear();
    // Add providers here
})
```

### <a name="consume-configuration-during-app-startup"></a>Utilizzare la configurazione durante l'avvio dell'app

La configurazione fornita all'app in `ConfigureAppConfiguration` è disponibile durante l'avvio dell'app, incluso `Startup.ConfigureServices`. Per altre informazioni, vedere la sezione [Accedere alla configurazione durante l'avvio](#access-configuration-during-startup).

## <a name="command-line-configuration-provider"></a>Provider di configurazione della riga di comando

<xref:Microsoft.Extensions.Configuration.CommandLine.CommandLineConfigurationProvider> carica la configurazione da coppie chiave-valore di argomenti della riga di comando in fase di esecuzione.

Per attivare la configurazione della riga di comando, metodo di estensione <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> viene chiamato su un'istanza di <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder>.

`AddCommandLine` viene chiamato automaticamente quando viene chiamato il metodo `CreateDefaultBuilder(string [])`. Per altre informazioni, vedere la sezione [Configurazione predefinita](#default-configuration).

`CreateDefaultBuilder` carica anche:

* Una configurazione facoltativa dai file *appsettings.json* e *appsettings.{Environment}.json*.
* [Segreti utente (Secret Manager)](xref:security/app-secrets) nell'ambiente di sviluppo.
* Variabili di ambiente.

`CreateDefaultBuilder` aggiunge il provider di configurazione della riga di comando per ultimo. Gli argomenti della riga di comando passati in fase di esecuzione sostituiscono la configurazione impostata dagli altri provider.

`CreateDefaultBuilder` opera durante la creazione dell'host. Pertanto, la configurazione della riga di comando attivata da `CreateDefaultBuilder` può influire sul modo in cui l'host viene configurato.

Per le app basate sui modelli di ASP.NET Core, `AddCommandLine` è già stato chiamato da `CreateDefaultBuilder`. Per aggiungere altri provider di configurazione e mantenere la possibilità di sostituire la configurazione di tali provider con argomenti della riga di comando, chiamare i provider aggiuntivi dell'app in `ConfigureAppConfiguration` e quindi chiamare `AddCommandLine` per ultimo.

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    // Call other providers here
    config.AddCommandLine(args);
})
```

**Esempio**

L'app di esempio consente di sfruttare il metodo di servizio statico `CreateDefaultBuilder` per creare l'host, che include una chiamata a <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*>.

1. Aprire un prompt dei comandi nella directory del progetto.
1. Fornire un argomento della riga di comando per il comando `dotnet run`, `dotnet run CommandLineKey=CommandLineValue`.
1. Dopo che l'app è in esecuzione, aprire un browser per l'app all'indirizzo `http://localhost:5000`.
1. Notare che l'output contiene la coppia chiave-valore per l'argomento della riga di comando di configurazione fornito per `dotnet run`.

### <a name="arguments"></a>Argomenti

Il valore deve seguire un segno di uguale (`=`) o la chiave deve avere un prefisso (`--` o `/`) quando il valore segue uno spazio. Il valore non è necessario se viene usato un segno di uguale, ad esempio `CommandLineKey=`.

| Prefisso chiave               | Esempio                                                |
| ---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

------------ | Titolo---: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

--------------------------- | | Nessun prefisso | `CommandLineKey1=value1`                               |
| Due trattini ( `--` ) | `--CommandLineKey2=value2` , `--CommandLineKey2 value2` |
 | Barra ( `/` ) | `/CommandLineKey3=value3` ,`/CommandLineKey3 value3`   |

Nello stesso comando, non mischiare coppie chiave-valore di argomenti della riga di comando che usano un segno di uguale con coppie chiave-valore che usano uno spazio.

Comandi di esempio:

```dotnetcli
dotnet run CommandLineKey1=value1 --CommandLineKey2=value2 /CommandLineKey3=value3
dotnet run --CommandLineKey1 value1 /CommandLineKey2 value2
dotnet run CommandLineKey1= CommandLineKey2=value2
```

### <a name="switch-mappings"></a>Mapping di sostituzione

I mapping di sostituzione consentono di usare la logica di sostituzione del nome della chiave. Quando si compila manualmente la configurazione con un oggetto <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> , fornire un dizionario di sostituzioni switch al <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> metodo.

Quando viene utilizzato il dizionario dei mapping di sostituzione, nel dizionario viene controllata la presenza di una chiave corrispondente alla chiave fornita da un argomento della riga di comando. Se la chiave della riga di comando viene trovata nel dizionario, il valore del dizionario (la sostituzione della chiave) viene passato nuovamente per impostare la coppia chiave-valore nella configurazione dell'app. Un mapping di sostituzione è necessario per le chiavi della riga di comando con un trattino singolo (`-`) come prefisso.

Regole principali del dizionario dei mapping di sostituzione:

* Le sostituzioni devono iniziare con un trattino (`-`) o un trattino doppio (`--`).
* Il dizionario dei mapping di sostituzione non deve contenere chiavi duplicate.

Creare un dizionario dei mapping di sostituzione. Nell'esempio seguente vengono creati due mapping di sostituzione:

```csharp
public static readonly Dictionary<string, string> _switchMappings = 
    new Dictionary<string, string>
    {
        { "-CLKey1", "CommandLineKey1" },
        { "-CLKey2", "CommandLineKey2" }
    };
```

Quando viene creato l'host, chiamare `AddCommandLine` con il dizionario dei mapping di sostituzione:

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddCommandLine(args, _switchMappings);
})
```

Per le app che usano i mapping di sostituzione, la chiamata a `CreateDefaultBuilder` non deve passare argomenti. La chiamata `AddCommandLine` del metodo `CreateDefaultBuilder` non include sostituzioni mappate e non è possibile passare il dizionario dei mapping di sostituzione a `CreateDefaultBuilder`. La soluzione consiste nel non passare gli argomenti a `CreateDefaultBuilder` consentendo invece al metodo `AddCommandLine` del metodo `ConfigurationBuilder` di elaborare entrambi gli argomenti e il dizionario dei mapping di sostituzione.

Il dizionario dei mapping di sostituzione creato contiene i dati visualizzati nella tabella seguente.

| Chiave       | Valore             |
| ---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

----- | Titolo---: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

--------- | | `-CLKey1` | `CommandLineKey1` |
| `-CLKey2` | `CommandLineKey2` |

Se le chiavi con mapping di sostituzione vengono usate all'avvio dell'app, la configurazione riceve il valore di configurazione per la chiave fornita dal dizionario:

```dotnetcli
dotnet run -CLKey1=value1 -CLKey2=value2
```

Dopo aver eseguito il comando precedente, la configurazione contiene i valori mostrati nella tabella seguente.

| Chiave               | Valore    |
| ---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

--------- | Titolo---: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

---- | | `CommandLineKey1` | `value1` |
| `CommandLineKey2` | `value2` |

## <a name="environment-variables-configuration-provider"></a>Provider di configurazione delle variabili di ambiente

<xref:Microsoft.Extensions.Configuration.EnvironmentVariables.EnvironmentVariablesConfigurationProvider> carica la configurazione da coppie chiave-valore di variabili di ambiente in fase di esecuzione.

Per attivare la configurazione delle variabili di ambiente, chiamare il metodo di estensione <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> su un'istanza di <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder>.

[!INCLUDE[](~/includes/environmentVarableColon.md)]

[App Azure servizio](https://azure.microsoft.com/services/app-service/) consente di impostare le variabili di ambiente nel portale di Azure che possono eseguire l'override della configurazione dell'app usando il provider di configurazione delle variabili di ambiente. Per altre informazioni, vedere [App di Azure: Eseguire l'override della configurazione delle app usando il portale di Azure](xref:host-and-deploy/azure-apps/index#override-app-configuration-using-the-azure-portal).

`AddEnvironmentVariables` consente di caricare variabili di ambiente con prefisso `ASPNETCORE_` per la [configurazione host](#host-versus-app-configuration) quando viene inizializzato un nuovo generatore di host con l'[host Web](xref:fundamentals/host/web-host) e viene chiamato `CreateDefaultBuilder`. Per altre informazioni, vedere la sezione [Configurazione predefinita](#default-configuration).

`CreateDefaultBuilder` carica anche:

* Configurazione delle app dalle variabili di ambiente senza prefisso chiamando `AddEnvironmentVariables` senza prefisso.
* Una configurazione facoltativa dai file *appsettings.json* e *appsettings.{Environment}.json*.
* [Segreti utente (Secret Manager)](xref:security/app-secrets) nell'ambiente di sviluppo.
* Argomenti della riga di comando.

Il provider di configurazione delle variabili di ambiente viene chiamato dopo aver stabilito la configurazione dai segreti utente e dai file *appsettings*. La chiamata del provider in questa posizione consente alle variabili di ambiente lette in fase di esecuzione di sostituire la configurazione impostata dai segreti utente e dai file *appsettings*.

Per fornire la configurazione dell'app da altre variabili di ambiente, chiamare i provider aggiuntivi dell'app in `ConfigureAppConfiguration` e chiamare `AddEnvironmentVariables` con il prefisso:

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddEnvironmentVariables(prefix: "PREFIX_");
})
```

Chiamare `AddEnvironmentVariables` Last per consentire alle variabili di ambiente con il prefisso specificato di eseguire l'override dei valori di altri provider.

**Esempio**

L'app di esempio consente di sfruttare il metodo di servizio statico `CreateDefaultBuilder` per creare l'host, che include una chiamata a `AddEnvironmentVariables`.

1. Eseguire l'app di esempio. Aprire un browser per l'app all'indirizzo `http://localhost:5000`.
1. Notare che l'output contiene la coppia chiave-valore per la variabile di ambiente `ENVIRONMENT`. Il valore riflette l'ambiente in cui l'app è in esecuzione, in genere `Development` durante l'esecuzione locale.

Per limitare l'elenco delle variabili di ambiente restituito dall'app, l'app filtra le variabili di ambiente. Vedere il file *Pages/Index.cshtml.cs* dell'app di esempio.

Per esporre tutte le variabili di ambiente disponibili per l'app, modificare `FilteredConfiguration` in *pages/index. cshtml. cs* come riportato di seguito:

```csharp
FilteredConfiguration = _config.AsEnumerable();
```

### <a name="prefixes"></a>Prefissi

Le variabili di ambiente caricate nella configurazione dell'app vengono filtrate quando si specifica un prefisso per il `AddEnvironmentVariables` metodo. Ad esempio, per filtrare le variabili di ambiente in base al prefisso `CUSTOM_`, fornire il prefisso al provider di configurazione:

```csharp
var config = new ConfigurationBuilder()
    .AddEnvironmentVariables("CUSTOM_")
    .Build();
```

Il prefisso viene rimosso quando vengono create le coppie chiave-valore della configurazione.

Quando viene creato il generatore di host, la configurazione dell'host viene fornita dalle variabili di ambiente. Per altre informazioni sul prefisso usato per queste variabili di ambiente, vedere la sezione [Configurazione predefinita](#default-configuration).

**Prefissi della stringa di connessione**

L'API di configurazione prevede regole di elaborazione speciali per quattro variabili di ambiente di stringa di connessione coinvolte nella configurazione di stringhe di connessione di Azure per l'ambiente dell'app. Le variabili di ambiente con i prefissi indicati nella tabella vengono caricate nell'app se non viene fornito alcun prefisso a `AddEnvironmentVariables`.

| Prefisso della stringa di connessione | Provider |
| ---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

------------ | Titolo---: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

---- | | `CUSTOMCONNSTR_` | Provider personalizzato | | `MYSQLCONNSTR_` | [MySQL](https://www.mysql.com/) |
 MySQL | `SQLAZURECONNSTR_` | SQL Server del [database SQL di Azure](https://azure.microsoft.com/services/sql-database/) |
 | `SQLCONNSTR_` | [SQL Server](https://www.microsoft.com/sql-server/)|

Quando una variabile di ambiente viene individuata e caricata nella configurazione con uno qualsiasi dei quattro prefissi indicati nella tabella:

* La chiave di configurazione viene creata rimuovendo il prefisso della variabile di ambiente e aggiungendo una sezione per la chiave di configurazione (`ConnectionStrings`).
* Viene creata una nuova coppia chiave-valore della configurazione che rappresenta il provider di connessione al database (ad eccezione di `CUSTOMCONNSTR_`, che non ha un provider dichiarato).

| Chiave della variabile di ambiente | Chiave di configurazione convertita | Voce di configurazione del provider                                                    |
| ---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

------------ | Titolo---: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-------------- | Titolo---: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

---------------------------------------- | | `CUSTOMCONNSTR_{KEY} `   | `ConnectionStrings:{KEY}`   | Voce di configurazione non creata.                                                | | `MYSQLCONNSTR_{KEY}`     | `ConnectionStrings:{KEY}`   | Chiave: `ConnectionStrings:{KEY}_ProviderName` :<br>Valore: `MySql.Data.MySqlClient` | | `SQLAZURECONNSTR_{KEY}`   |  `ConnectionStrings:{KEY}`   | Chiave: `ConnectionStrings:{KEY}_ProviderName` :<br>Valore: `System.Data.SqlClient` | | `SQLCONNSTR_{KEY}`        |  `ConnectionStrings:{KEY}`   | Chiave: `ConnectionStrings:{KEY}_ProviderName` :<br>Valore`System.Data.SqlClient`  |

**Esempio**

Nel server viene creata una variabile di ambiente della stringa di connessione personalizzata:

* Nome: `CUSTOMCONNSTR_ReleaseDB`
* Valore: `Data Source=ReleaseSQLServer;Initial Catalog=MyReleaseDB;Integrated Security=True`

Se `IConfiguration` viene inserito e assegnato a un campo denominato `_config` , leggere il valore:

```csharp
_config["ConnectionStrings:ReleaseDB"]
```

## <a name="file-configuration-provider"></a>Provider di configurazione dei file

<xref:Microsoft.Extensions.Configuration.FileConfigurationProvider> è la classe base per il caricamento della configurazione dal file system. I provider di configurazione seguenti sono dedicati per specifici tipi di file:

* [Provider di configurazione INI](#ini-configuration-provider)
* [Provider di configurazione JSON](#json-configuration-provider)
* [Provider di configurazione XML](#xml-configuration-provider)

### <a name="ini-configuration-provider"></a>Provider di configurazione INI

<xref:Microsoft.Extensions.Configuration.Ini.IniConfigurationProvider> carica la configurazione da coppie chiave-valore di file INI in fase di esecuzione.

Per attivare la configurazione di file INI, chiamare il metodo di estensione <xref:Microsoft.Extensions.Configuration.IniConfigurationExtensions.AddIniFile*> su un'istanza di <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder>.

I due punti possono essere usati come delimitatore di sezione nella configurazione di file INI.

Gli overload consentono di specificare:

* Se il file è facoltativo.
* Se la configurazione viene ricaricata se viene modificato il file.
* Il <xref:Microsoft.Extensions.FileProviders.IFileProvider> usato per accedere al file.

Chiamare `ConfigureAppConfiguration` quando si crea l'host per specificare la configurazione dell'app:

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddIniFile(
        "config.ini", optional: true, reloadOnChange: true);
})
```

Un esempio generico di un file di configurazione INI:

```ini
[section0]
key0=value
key1=value

[section1]
subsection:key=value

[section2:subsection0]
key=value

[section2:subsection1]
key=value
```

Il file di configurazione precedente carica le chiavi seguenti con `value`:

* section0:key0
* section0:key1
* section1:subsection:key
* section2:subsection0:key
* section2:subsection1:key

### <a name="json-configuration-provider"></a>Provider di configurazione JSON

Il <xref:Microsoft.Extensions.Configuration.Json.JsonConfigurationProvider> carica la configurazione da coppie chiave-valore di file JSON in fase di esecuzione.

Per attivare la configurazione di file JSON, chiamare il metodo di estensione <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> su un'istanza di <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder>.

Gli overload consentono di specificare:

* Se il file è facoltativo.
* Se la configurazione viene ricaricata se viene modificato il file.
* Il <xref:Microsoft.Extensions.FileProviders.IFileProvider> usato per accedere al file.

`AddJsonFile`viene chiamato automaticamente due volte quando viene inizializzato un nuovo generatore host con `CreateDefaultBuilder` . Il metodo viene chiamato per caricare la configurazione da:

* *appSettings. JSON*: questo file viene letto per primo. La versione dell'ambiente del file può sostituire i valori forniti dal file *appsettings.json*.
* *appSettings. {Environment}. JSON*: la versione dell'ambiente del file è stata caricata in base a [IHostingEnvironment. EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.EnvironmentName*).

Per altre informazioni, vedere la sezione [Configurazione predefinita](#default-configuration).

`CreateDefaultBuilder` carica anche:

* Variabili di ambiente.
* [Segreti utente (Secret Manager)](xref:security/app-secrets) nell'ambiente di sviluppo.
* Argomenti della riga di comando.

Il provider di configurazione JSON viene stabilito per primo. I segreti utente, le variabili di ambiente e gli argomenti della riga di comando sostituiscono quindi la configurazione impostata dai file *appsettings*.

Chiamare `ConfigureAppConfiguration` quando si crea l'host per specificare la configurazione dell'app per file diversi da *appsettings.json* e *appsettings.{Environment}.json*:

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddJsonFile(
        "config.json", optional: true, reloadOnChange: true);
})
```

**Esempio**

L'app di esempio sfrutta il metodo di convenienza statica `CreateDefaultBuilder` per compilare l'host, che include due chiamate a `AddJsonFile` :

* La prima chiamata a `AddJsonFile` carica la configurazione da *appSettings. JSON*:

  [!code-json[](index/samples/2.x/ConfigurationSample/appsettings.json)]

* La seconda chiamata a `AddJsonFile` carica la configurazione da *appSettings. { Environment}. JSON*. Per *appSettings. Development. JSON* nell'app di esempio, viene caricato il file seguente:

  [!code-json[](index/samples/2.x/ConfigurationSample/appsettings.Development.json)]

1. Eseguire l'app di esempio. Aprire un browser per l'app all'indirizzo `http://localhost:5000`.
1. L'output contiene coppie chiave-valore per la configurazione basata sull'ambiente dell'app. Il livello di registrazione della chiave `Logging:LogLevel:Default` è `Debug` quando si esegue l'app nell'ambiente di sviluppo.
1. Eseguire di nuovo l'app di esempio nell'ambiente di produzione:
   1. Aprire il file *Properties/launchSettings. JSON* .
   1. Nel `ConfigurationSample` profilo, modificare il valore della variabile di `ASPNETCORE_ENVIRONMENT` ambiente in `Production` .
   1. Salvare il file ed eseguire l'app con `dotnet run` in una shell dei comandi.
1. Impostazioni in *appSettings. Development. JSON* non sostituisce più le impostazioni in *appSettings. JSON*. Il livello di registrazione per la chiave `Logging:LogLevel:Default` è `Warning` .

### <a name="xml-configuration-provider"></a>Provider di configurazione XML

Il <xref:Microsoft.Extensions.Configuration.Xml.XmlConfigurationProvider> carica la configurazione da coppie chiave-valore di file XML in fase di esecuzione.

Per attivare la configurazione di file XML, chiamare il metodo di estensione <xref:Microsoft.Extensions.Configuration.XmlConfigurationExtensions.AddXmlFile*> su un'istanza di <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder>.

Gli overload consentono di specificare:

* Se il file è facoltativo.
* Se la configurazione viene ricaricata se viene modificato il file.
* Il <xref:Microsoft.Extensions.FileProviders.IFileProvider> usato per accedere al file.

Il nodo radice del file di configurazione viene ignorato quando vengono create le coppie chiave-valore della configurazione. Non specificare una definizione DTD (Document Type Definition) o uno spazio dei nomi nel file.

Chiamare `ConfigureAppConfiguration` quando si crea l'host per specificare la configurazione dell'app:

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddXmlFile(
        "config.xml", optional: true, reloadOnChange: true);
})
```

I file di configurazione XML possono usare nomi di elementi distinti per le sezioni ripetute:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <section0>
    <key0>value</key0>
    <key1>value</key1>
  </section0>
  <section1>
    <key0>value</key0>
    <key1>value</key1>
  </section1>
</configuration>
```

Il file di configurazione precedente carica le chiavi seguenti con `value`:

* section0:key0
* section0:key1
* section1:key0
* section1:key1

La ripetizione di elementi che usano lo stesso nome di elemento funziona se si usa l'attributo `name` per distinguere gli elementi:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <section name="section0">
    <key name="key0">value</key>
    <key name="key1">value</key>
  </section>
  <section name="section1">
    <key name="key0">value</key>
    <key name="key1">value</key>
  </section>
</configuration>
```

Il file di configurazione precedente carica le chiavi seguenti con `value`:

* section:section0:key:key0
* section:section0:key:key1
* section:section1:key:key0
* section:section1:key:key1

È possibile usare attributi per fornire valori:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <key attribute="value" />
  <section>
    <key attribute="value" />
  </section>
</configuration>
```

Il file di configurazione precedente carica le chiavi seguenti con `value`:

* key:attribute
* section:key:attribute

## <a name="key-per-file-configuration-provider"></a>Provider di configurazione KeyPerFile

Il <xref:Microsoft.Extensions.Configuration.KeyPerFile.KeyPerFileConfigurationProvider> usa i file di una directory come coppie chiave-valore della configurazione. La chiave è il nome del file. Il valore contiene il contenuto del file. Il provider di configurazione KeyPerFile viene usato in scenari di hosting Docker.

Per attivare la configurazione chiave-per-file, chiamare il metodo di estensione <xref:Microsoft.Extensions.Configuration.KeyPerFileConfigurationBuilderExtensions.AddKeyPerFile*> su un'istanza di <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder>. Il `directoryPath` per i file deve essere un percorso assoluto.

Gli overload consentono di specificare:

* Un delegato `Action<KeyPerFileConfigurationSource>` che configura l'origine.
* Se la directory è facoltativa e il percorso della directory.

Il doppio carattere di sottolineatura (`__`) viene usato come delimitatore per le chiavi di configurazione nei nomi dei file. Ad esempio, il nome di file `Logging__LogLevel__System` produce la chiave di configurazione `Logging:LogLevel:System`.

Chiamare `ConfigureAppConfiguration` quando si crea l'host per specificare la configurazione dell'app:

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    var path = Path.Combine(
        Directory.GetCurrentDirectory(), "path/to/files");
    config.AddKeyPerFile(directoryPath: path, optional: true);
})
```

## <a name="memory-configuration-provider"></a>Provider di configurazione della memoria

Il <xref:Microsoft.Extensions.Configuration.Memory.MemoryConfigurationProvider> usa una raccolta in memoria come coppie chiave-valore della configurazione.

Per attivare la configurazione della raccolta in memoria, chiamare il metodo di estensione <xref:Microsoft.Extensions.Configuration.MemoryConfigurationBuilderExtensions.AddInMemoryCollection*> su un'istanza di <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder>.

Il provider di configurazione può essere inizializzato con un `IEnumerable<KeyValuePair<String,String>>`.

Chiamare `ConfigureAppConfiguration` quando si crea l'host per specificare la configurazione dell'app.

Nell'esempio seguente viene creato un dizionario di configurazione:

```csharp
public static readonly Dictionary<string, string> _dict = 
    new Dictionary<string, string>
    {
        {"MemoryCollectionKey1", "value1"},
        {"MemoryCollectionKey2", "value2"}
    };
```

Il dizionario viene usato con una chiamata a `AddInMemoryCollection` per fornire la configurazione:

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddInMemoryCollection(_dict);
})
```

## <a name="getvalue"></a>GetValue

[`ConfigurationBinder.GetValue<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.GetValue*)estrae un singolo valore dalla configurazione con una chiave specificata e lo converte nel tipo non di raccolta specificato. Un overload accetta un valore predefinito.

L'esempio seguente:

* Estrae il valore di stringa dalla configurazione con la chiave `NumberKey`. Se `NumberKey` non viene trovato nelle chiavi di configurazione, viene usato il valore predefinito di `99`.
* Assegna al valore il tipo `int`.
* Memorizza il valore nella proprietà `NumberConfig` per l'uso da parte della pagina.

```csharp
public class IndexModel : PageModel
{
    public IndexModel(IConfiguration config)
    {
        _config = config;
    }

    public int NumberConfig { get; private set; }

    public void OnGet()
    {
        NumberConfig = _config.GetValue<int>("NumberKey", 99);
    }
}
```

## <a name="getsection-getchildren-and-exists"></a>GetSection, GetChildren ed Exists

Per gli esempi che seguono, prendere in considerazione il file JSON seguente. Vengono trovate quattro chiavi in due sezioni, una delle quali include una coppia di sottosezioni:

```json
{
  "section0": {
    "key0": "value",
    "key1": "value"
  },
  "section1": {
    "key0": "value",
    "key1": "value"
  },
  "section2": {
    "subsection0" : {
      "key0": "value",
      "key1": "value"
    },
    "subsection1" : {
      "key0": "value",
      "key1": "value"
    }
  }
}
```

Quando il file viene letto nella configurazione, vengono create le chiavi gerarchiche univoche seguenti per contenere i valori di configurazione:

* section0:key0
* section0:key1
* section1:key0
* section1:key1
* section2:subsection0:key0
* section2:subsection0:key1
* section2:subsection1:key0
* section2:subsection1:key1

### <a name="getsection"></a>GetSection

[IConfiguration.GetSection](xref:Microsoft.Extensions.Configuration.IConfiguration.GetSection*) estrae una sottosezione della configurazione con la chiave di sottosezione specificata.

Per restituire una <xref:Microsoft.Extensions.Configuration.IConfigurationSection> che contiene solo le coppie chiave-valore in `section1`, chiamare `GetSection` e fornire il nome della sezione:

```csharp
var configSection = _config.GetSection("section1");
```

`configSection` non ha un valore, ma solo una chiave e un percorso.

In modo analogo, per ottenere i valori per le chiavi in `section2:subsection0`, chiamare `GetSection` e fornire il percorso della sezione:

```csharp
var configSection = _config.GetSection("section2:subsection0");
```

`GetSection` non restituisce mai `null`. Se non viene trovata una sezione corrispondente, viene restituita una `IConfigurationSection` vuota.

Quando `GetSection` restituisce una sezione corrispondente, <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Value> non viene compilata. Quando la sezione esiste, vengono restituiti <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Key> e <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Path>.

### <a name="getchildren"></a>GetChildren

Una chiamata a [IConfiguration.GetChildren](xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*) per `section2` ottiene una `IEnumerable<IConfigurationSection>` che include:

* `subsection0`
* `subsection1`

```csharp
var configSection = _config.GetSection("section2");

var children = configSection.GetChildren();
```

### <a name="exists"></a>Exists

Usare [ConfigurationExtensions.Exists](xref:Microsoft.Extensions.Configuration.ConfigurationExtensions.Exists*) per determinare se esiste una sezione di configurazione:

```csharp
var sectionExists = _config.GetSection("section2:subsection2").Exists();
```

Considerati i dati di esempio, `sectionExists` è `false` perché non esiste una sezione `section2:subsection2` nei dati di configurazione.

## <a name="bind-to-an-object-graph"></a>Associazione a un oggetto grafico

<xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*> è in grado di associare un intero oggetto grafico POCO. Come per l'associazione di un oggetto semplice, vengono associate solo le proprietà di lettura/scrittura pubbliche.

L'esempio contiene un modello `TvShow` il cui oggetto grafico include `Metadata` e le classi `Actors` (*Models/TvShow.cs*):

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/TvShow.cs?name=snippet1)]

L'app di esempio ha un file *tvshow.xml* che contiene i dati di configurazione:

[!code-xml[](index/samples/2.x/ConfigurationSample/tvshow.xml)]

La configurazione viene associata all'intero oggetto grafico `TvShow` con il metodo `Bind`. L'istanza associata viene assegnata a una proprietà per il rendering:

```csharp
var tvShow = new TvShow();
_config.GetSection("tvshow").Bind(tvShow);
TvShow = tvShow;
```

[`ConfigurationBinder.Get<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Get*)associa e restituisce il tipo specificato. `Get<T>` può essere più comodo che usare `Bind`. Il codice seguente illustra come usare `Get<T>` con l'esempio precedente:

[!code-csharp[](index/samples/2.x/ConfigurationSample/Pages/Index.cshtml.cs?name=snippet_tvshow)]

## <a name="bind-an-array-to-a-class"></a>Associare una matrice a una classe

*L'app di esempio dimostra i concetti spiegati in questa sezione.*

Il <xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*> supporta l'associazione di matrici agli oggetti usando gli indici delle matrici nelle chiavi di configurazione. Qualsiasi formato di matrice che espone un segmento di chiave numerico ( `:0:` , `:1:` , &hellip; `:{n}:` ) è in grado di associare array a una matrice di classi poco.

> [!NOTE]
> L'associazione viene fornita per convenzione. I provider di configurazione personalizzati non devono implementare l'associazione di matrici.

**Elaborazione di matrici in memoria**

Prendere in considerazione le chiavi di configurazione e i valori indicati nella tabella seguente.

| Chiave             | Valore  |
| :---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-------: | :----: | | matrice: voci: 0 | value0 | | matrice: voci: 1 | value1 | | matrice: voci: 2 | Value2 | | matrice: voci: 4 | Value4 | | matrice: voci: 5 | value5 |

Queste chiavi e i valori vengono caricati nell'app di esempio usando il provider di configurazione della memoria:

[!code-csharp[](index/samples/2.x/ConfigurationSample/Program.cs?name=snippet_Program&highlight=5-12,22)]

La matrice ignora un valore per l'indice &num;3. Lo strumento di associazione di configurazione non è in grado di associare valori null o di creare voci null negli oggetti associati, situazione che diventerà chiara tra poco quando viene illustrato il risultato dell'associazione di questa matrice a un oggetto.

Nell'app di esempio, è disponibile una classe POCO per i dati di configurazione associati:

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/ArrayExample.cs?name=snippet1)]

I dati di configurazione sono associati all'oggetto:

```csharp
var arrayExample = new ArrayExample();
_config.GetSection("array").Bind(arrayExample);
```

[`ConfigurationBinder.Get<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Get*)è anche possibile usare la sintassi, che comporta un codice più compatto:

[!code-csharp[](index/samples/2.x/ConfigurationSample/Pages/Index.cshtml.cs?name=snippet_array)]

L'oggetto associato, un'istanza di `ArrayExample`, riceve i dati di matrice dalla configurazione.

| Indice di `ArrayExample.Entries` | `ArrayExample.Entries` Valore |
| :---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-------------: | :---title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-------------: | | 0 | value0 | | 1 | value1 | | 2 | Value2 | | 3 | Value4 | | 4 | value5 |

L'indice &num;3 nell'oggetto associato contiene i dati di configurazione per la chiave di configurazione `array:4` e il relativo valore `value4`. Quando i dati di configurazione contenenti una matrice vengono associati, gli indici di matrice nelle chiavi di configurazione vengono usati semplicemente per scorrere i dati di configurazione quando si crea l'oggetto. Un valore null non può essere mantenuto nei dati di configurazione e una voce con valore null non viene creata in un oggetto associato quando una matrice nelle chiavi di configurazione ignora uno o più indici.

L'elemento di configurazione mancante per l'indice &num;3 può essere fornito prima dell'associazione all'istanza `ArrayExample` da qualsiasi provider di configurazione che produce la coppia chiave-valore corretta nella configurazione. Se il codice di esempio include un provider di configurazione JSON aggiuntivo con la coppia chiave-valore mancante, `ArrayExample.Entries` corrisponde alla matrice di configurazione completa:

*missing_value.json*:

```json
{
  "array:entries:3": "value3"
}
```

In `ConfigureAppConfiguration`:

```csharp
config.AddJsonFile(
    "missing_value.json", optional: false, reloadOnChange: false);
```

La coppia chiave-valore mostrata nella tabella viene caricata nella configurazione.

| Chiave             | Valore  |
| :---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-------: | :----: | | matrice: voci: 3 | valore3 |

Se l'istanza della classe `ArrayExample` viene associata dopo che il provider di configurazione JSON include la voce per l'indice &num;3, la matrice `ArrayExample.Entries` include il valore.

| Indice di `ArrayExample.Entries` | `ArrayExample.Entries` Valore |
| :---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-------------: | :---title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-------------: | | 0 | value0 | | 1 | value1 | | 2 | Value2 | | 3 | valore3 | | 4 | Value4 | | 5 | value5 |

**Elaborazione di matrice JSON**

Se un file JSON contiene una matrice, vengono create chiavi di configurazione per gli elementi della matrice con un indice di sezione in base zero. Nel file di configurazione seguente, `subsection` è una matrice:

[!code-json[](index/samples/2.x/ConfigurationSample/json_array.json)]

Il provider di configurazione JSON legge i dati di configurazione nelle coppie chiave-valore seguenti:

| Chiave                     | Valore  |
| ---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

------------ | :----: | | json_array: chiave | valore a | | json_array: sottosezione: 0 | valoreB | | json_array: sottosezione: 1 | Valorec | | json_array: sottosezione: 2 | con valori |

Nell'app di esempio, la classe POCO seguente è disponibile per associare le coppie chiave-valore della configurazione:

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/JsonArrayExample.cs?name=snippet1)]

Dopo l'associazione, `JsonArrayExample.Key` contiene il valore `valueA`. I valori di sottosezione vengono archiviati nella proprietà matrice POCO `Subsection`.

| Indice di `JsonArrayExample.Subsection` | `JsonArrayExample.Subsection` Valore |
| :---
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-----------------: | :---title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: autore: Descrizione: monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-----------------: | | 0 | valoreB | | 1 | Valorec | | 2 | con valori |

## <a name="custom-configuration-provider"></a>Provider di configurazione personalizzato

L'app di esempio dimostra come creare un provider di configurazione di base che legge le coppie chiave-valore di configurazione da un database usando [Entity Framework (EF)](/ef/core/).

Il provider ha le caratteristiche seguenti:

* Il database in memoria di Entity Framework viene usato a scopo dimostrativo. Per usare un database che richiede una stringa di connessione, implementare un `ConfigurationBuilder` secondario per fornire la stringa di connessione da un altro provider di configurazione.
* Il provider legge una tabella di database in una configurazione all'avvio. Il provider non esegue una query sul database per ogni chiave.
* Il ricaricamento in caso di modifica non è implementato, quindi l'aggiornamento del database dopo l'avvio dell'app non ha alcun effetto sulla configurazione dell'app.

Definire un'entità `EFConfigurationValue` per l'archiviazione dei valori di configurazione nel database.

*Models/EFConfigurationValue.cs*:

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/EFConfigurationValue.cs?name=snippet1)]

Aggiungere `EFConfigurationContext` per archiviare i valori configurati e accedervi.

*EFConfigurationProvider/EFConfigurationContext.cs*:

[!code-csharp[](index/samples/2.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationContext.cs?name=snippet1)]

Creare una classe che implementi <xref:Microsoft.Extensions.Configuration.IConfigurationSource>.

*EFConfigurationProvider/EFConfigurationSource.cs*:

[!code-csharp[](index/samples/2.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationSource.cs?name=snippet1)]

Creare il provider di configurazione personalizzato ereditando da <xref:Microsoft.Extensions.Configuration.ConfigurationProvider>. Il provider di configurazione inizializza il database quando è vuoto.

*EFConfigurationProvider/EFConfigurationProvider.cs*:

[!code-csharp[](index/samples/2.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationProvider.cs?name=snippet1)]

Un metodo di estensione `AddEFConfiguration` consente di aggiungere l'origine di configurazione a un `ConfigurationBuilder`.

*Extensions/EntityFrameworkExtensions.cs*:

[!code-csharp[](index/samples/2.x/ConfigurationSample/Extensions/EntityFrameworkExtensions.cs?name=snippet1)]

L'esempio di codice seguente mostra come usare il `EFConfigurationProvider` personalizzato in *Program.cs*:

[!code-csharp[](index/samples/2.x/ConfigurationSample/Program.cs?name=snippet_Program&highlight=29-30)]

## <a name="access-configuration-during-startup"></a>Accedere alla configurazione durante l'avvio

Inserire `IConfiguration` nel costruttore `Startup` per accedere ai valori di configurazione in `Startup.ConfigureServices`. Per accedere alla configurazione in `Startup.Configure`, inserire `IConfiguration` direttamente nel metodo o usare l'istanza dal costruttore:

```csharp
public class Startup
{
    private readonly IConfiguration _config;

    public Startup(IConfiguration config)
    {
        _config = config;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        var value = _config["key"];
    }

    public void Configure(IApplicationBuilder app, IConfiguration config)
    {
        var value = config["key"];
    }
}
```

Per un esempio di accesso alla configurazione usando metodi di servizio di avvio, vedere [Avvio dell'applicazione: Metodi pratici](xref:fundamentals/startup#convenience-methods).

## <a name="access-configuration-in-a-razor-pages-page-or-mvc-view"></a>Accedere alla configurazione in una Razor pagina di pagine o in una visualizzazione MVC

Per accedere alle impostazioni di configurazione in una Razor pagina di pagine o in una visualizzazione MVC, aggiungere una [direttiva using](xref:mvc/views/razor#using) ([riferimenti per C#: direttiva using](/dotnet/csharp/language-reference/keywords/using-directive)) per lo [spazio dei nomi Microsoft. Extensions. Configuration](xref:Microsoft.Extensions.Configuration) e inserire <xref:Microsoft.Extensions.Configuration.IConfiguration> nella pagina o nella vista.

In una Razor pagina di pagine:

```cshtml
@page
@model IndexModel
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<!DOCTYPE html>
<html lang="en">
<head>
    <title>Index Page</title>
</head>
<body>
    <h1>Access configuration in a Razor Pages page</h1>
    <p>Configuration value for 'key': @Configuration["key"]</p>
</body>
</html>
```

In una visualizzazione MVC:

```cshtml
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<!DOCTYPE html>
<html lang="en">
<head>
    <title>Index View</title>
</head>
<body>
    <h1>Access configuration in an MVC view</h1>
    <p>Configuration value for 'key': @Configuration["key"]</p>
</body>
</html>
```

## <a name="add-configuration-from-an-external-assembly"></a>Aggiungere la configurazione da un assembly esterno

Un'implementazione <xref:Microsoft.AspNetCore.Hosting.IHostingStartup> consente l'aggiunta di miglioramenti a un'app all'avvio, da un assembly esterno alla classe `Startup` dell'app. Per altre informazioni, vedere <xref:fundamentals/configuration/platform-specific-configuration>.

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:fundamentals/configuration/options>

::: moniker-end
