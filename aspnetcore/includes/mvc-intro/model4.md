<span data-ttu-id="49848-101">Nella tabella seguente sono specificati i parametri del generatore di codice ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="49848-101">The following table details the ASP.NET Core code generator parameters:</span></span>

| <span data-ttu-id="49848-102">Parametro</span><span class="sxs-lookup"><span data-stu-id="49848-102">Parameter</span></span>               | <span data-ttu-id="49848-103">Description</span><span class="sxs-lookup"><span data-stu-id="49848-103">Description</span></span>|
| ----------------- | ------------ |
| <span data-ttu-id="49848-104">-m</span><span class="sxs-lookup"><span data-stu-id="49848-104">-m</span></span>  | <span data-ttu-id="49848-105">Nome del modello.</span><span class="sxs-lookup"><span data-stu-id="49848-105">The name of the model.</span></span> |
| <span data-ttu-id="49848-106">-dc</span><span class="sxs-lookup"><span data-stu-id="49848-106">-dc</span></span>  | <span data-ttu-id="49848-107">Contesto dati.</span><span class="sxs-lookup"><span data-stu-id="49848-107">The data context.</span></span> |
| <span data-ttu-id="49848-108">-udl</span><span class="sxs-lookup"><span data-stu-id="49848-108">-udl</span></span> | <span data-ttu-id="49848-109">Uso del layout predefinito.</span><span class="sxs-lookup"><span data-stu-id="49848-109">Use the default layout.</span></span> |
| <span data-ttu-id="49848-110">--relativeFolderPath</span><span class="sxs-lookup"><span data-stu-id="49848-110">--relativeFolderPath</span></span> | <span data-ttu-id="49848-111">Percorso relativo della cartella di output per creare le viste.</span><span class="sxs-lookup"><span data-stu-id="49848-111">The relative output folder path to create the views.</span></span> |
| <span data-ttu-id="49848-112">--useDefaultLayout</span><span class="sxs-lookup"><span data-stu-id="49848-112">--useDefaultLayout</span></span> | <span data-ttu-id="49848-113">Per le viste deve essere usato il layout predefinito.</span><span class="sxs-lookup"><span data-stu-id="49848-113">The default layout should be used for the views.</span></span> |
| <span data-ttu-id="49848-114">--referenceScriptLibraries</span><span class="sxs-lookup"><span data-stu-id="49848-114">--referenceScriptLibraries</span></span> | <span data-ttu-id="49848-115">Aggiunge `_ValidationScriptsPartial` per modificare e creare pagine</span><span class="sxs-lookup"><span data-stu-id="49848-115">Adds `_ValidationScriptsPartial` to Edit and Create pages</span></span> |

<span data-ttu-id="49848-116">Utilizzare il commutatore `h` per ottenere assistenza sul comando `aspnet-codegenerator controller`:</span><span class="sxs-lookup"><span data-stu-id="49848-116">Use the `h` switch to get help on the `aspnet-codegenerator controller` command:</span></span>

```console
dotnet aspnet-codegenerator controller -h
```