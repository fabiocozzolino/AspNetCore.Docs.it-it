---
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

---
# <a name="configure-the-linker-for-aspnet-core-blazor"></a>Configurare il linker per ASP.NET CoreBlazor

Di [Luke Latham](https://github.com/guardrex)

BlazorWebassembly esegue il collegamento del [linguaggio intermedio (il)](/dotnet/standard/managed-code#intermediate-language--execution) durante una compilazione per eliminare il linguaggio intermedio non necessario dagli assembly di output dell'app. Il linker è disabilitato durante la compilazione nella configurazione di debug. Per abilitare il linker, le app devono essere compilate in configurazione versione. Quando si distribuiscono le app webassembly, è consigliabile compilare in versione Blazor . 

Il collegamento di un'app ottimizza le dimensioni, ma può avere effetti negativi. Le app che usano la reflection o le funzionalità dinamiche correlate potrebbero interrompersi quando vengono tagliate perché il linker non è a conoscenza di questo comportamento dinamico e non può determinare in generale quali tipi sono necessari per la reflection in fase di esecuzione. Per tagliare tali app, il linker deve essere informato sui tipi necessari per la reflection nel codice e nei pacchetti o Framework da cui dipende l'app. 

Per assicurarsi che l'app tagliata funzioni correttamente una volta distribuita, è importante testare le build di rilascio dell'app spesso durante lo sviluppo.

Il collegamento per Blazor le app può essere configurato usando le funzionalità di MSBuild seguenti:

* Configurare il collegamento a livello globale con una [proprietà di MSBuild](#control-linking-with-an-msbuild-property).
* Controllo del collegamento per ogni assembly con un [file di configurazione](#control-linking-with-a-configuration-file).

## <a name="control-linking-with-an-msbuild-property"></a>Controllo del collegamento con una proprietà MSBuild

Il collegamento viene abilitato quando si compila un'app nella `Release` configurazione. Per modificare questa configurazione, configurare la `BlazorWebAssemblyEnableLinking` Proprietà MSBuild nel file di progetto:

```xml
<PropertyGroup>
  <BlazorWebAssemblyEnableLinking>false</BlazorWebAssemblyEnableLinking>
</PropertyGroup>
```

## <a name="control-linking-with-a-configuration-file"></a>Controllare il collegamento con un file di configurazione

Controllare il collegamento per ogni singolo assembly usando un file di configurazione XML e specificando il file come un elemento MSBuild nel file di progetto:

```xml
<ItemGroup>
  <BlazorLinkerDescriptor Include="LinkerConfig.xml" />
</ItemGroup>
```

*LinkerConfig. XML*:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--
  This file specifies which parts of the BCL or Blazor packages must not be
  stripped by the IL Linker even if they aren't referenced by user code.
-->
<linker>
  <assembly fullname="mscorlib">
    <!--
      Preserve the methods in WasmRuntime because its methods are called by 
      JavaScript client-side code to implement timers.
      Fixes: https://github.com/dotnet/blazor/issues/239
    -->
    <type fullname="System.Threading.WasmRuntime" />
  </assembly>
  <assembly fullname="System.Core">
    <!--
      System.Linq.Expressions* is required by Json.NET and any 
      expression.Compile caller. The assembly isn't stripped.
    -->
    <type fullname="System.Linq.Expressions*" />
  </assembly>
  <!--
    In this example, the app's entry point assembly is listed. The assembly
    isn't stripped by the IL Linker.
  -->
  <assembly fullname="MyCoolBlazorApp" />
</linker>
```

Per altre informazioni ed esempi, vedere [formati di dati (repository GitHub mono/linker)](https://github.com/mono/linker/blob/master/docs/data-formats.md).

## <a name="add-an-xml-linker-configuration-file-to-a-library"></a>Aggiungere un file di configurazione del linker XML a una raccolta

Per configurare il linker per una libreria specifica, aggiungere un file di configurazione del linker XML nella libreria come risorsa incorporata. La risorsa incorporata deve avere lo stesso nome dell'assembly.

Nell'esempio seguente il file *LinkerConfig. XML* viene specificato come una risorsa incorporata con lo stesso nome dell'assembly della libreria:

```xml
<ItemGroup>
  <EmbeddedResource Include="LinkerConfig.xml">
    <LogicalName>$(MSBuildProjectName).xml</LogicalName>
  </EmbeddedResource>
</ItemGroup>
```

### <a name="configure-the-linker-for-internationalization"></a>Configurare il linker per l'internazionalizzazione

Per impostazione predefinita, Blazor la configurazione del linker per le Blazor app webassembly rimuove le informazioni di internazionalizzazione ad eccezione delle impostazioni locali richieste in modo esplicito. La rimozione di questi assembly riduce al minimo le dimensioni dell'app.

Per controllare quali assembly I18N vengono conservati, impostare la `<BlazorWebAssemblyI18NAssemblies>` Proprietà MSBuild nel file di progetto:

```xml
<PropertyGroup>
  <BlazorWebAssemblyI18NAssemblies>{all|none|REGION1,REGION2,...}</BlazorWebAssemblyI18NAssemblies>
</PropertyGroup>
```

| Valore Region     | Assembly dell'area mono    |
| ---
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-------- | Titolo---: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app."
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

-
title: "configurare il linker per ASP.NET Core Blazor " autore: Descrizione: "informazioni su come controllare il linker linguaggio intermedio (il) durante la compilazione di un' Blazor app".
monikerRange: ms. Author: ms. Custom: ms. Date: No-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- SignalRUID '': 

------------ | | `all`            | Tutti gli assembly inclusi | | `cjk`             |  *I18n. CJK. dll* | | `mideast`         |  *I18n. . Dll* `none`(impostazione predefinita) | Nessuno | | `other`           |  *I18n. Other. dll* | | `rare`            |  *I18n. Rare. dll* | | `west`            |  *I18n. Dll occidentale*         |

Usare una virgola per separare più valori (ad esempio, `mideast,west` ).

Per altre informazioni, vedere [i18n: Pnetlib internationalation Framework Library (repository GitHub mono/mono)](https://github.com/mono/mono/tree/master/mcs/class/I18N).

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:performance/blazor/webassembly-best-practices#intermediate-language-il-linking>
