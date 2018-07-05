---
title: Tipi restituiti per le azioni del controller nell'API Web di ASP.NET Core
author: scottaddie
description: Informazioni sull'uso dei vari tipi restituiti dal metodo per le azioni del controller nell'API Web di ASP.NET Core.
ms.author: scaddie
ms.custom: mvc
ms.date: 03/21/2018
uid: web-api/action-return-types
ms.openlocfilehash: 422db97da222fb5e742e1d8e6ae410edc90dbc18
ms.sourcegitcommit: a1afd04758e663d7062a5bfa8a0d4dca38f42afc
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 06/20/2018
ms.locfileid: "36273556"
---
# <a name="controller-action-return-types-in-aspnet-core-web-api"></a><span data-ttu-id="aa504-103">Tipi restituiti per le azioni del controller nell'API Web di ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="aa504-103">Controller action return types in ASP.NET Core Web API</span></span>

<span data-ttu-id="aa504-104">Di [Scott Addie](https://github.com/scottaddie)</span><span class="sxs-lookup"><span data-stu-id="aa504-104">By [Scott Addie](https://github.com/scottaddie)</span></span>

<span data-ttu-id="aa504-105">[Visualizzare o scaricare il codice di esempio](https://github.com/aspnet/Docs/tree/master/aspnetcore/web-api/action-return-types/samples) ([procedura per il download](xref:tutorials/index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="aa504-105">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/web-api/action-return-types/samples) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

<span data-ttu-id="aa504-106">ASP.NET Core offre le seguenti opzioni per i tipi restituiti per le azioni del controller nell'API Web:</span><span class="sxs-lookup"><span data-stu-id="aa504-106">ASP.NET Core offers the following options for Web API controller action return types:</span></span>

::: moniker range="<= aspnetcore-2.0"
* [<span data-ttu-id="aa504-107">Tipo specifico</span><span class="sxs-lookup"><span data-stu-id="aa504-107">Specific type</span></span>](#specific-type)
* [<span data-ttu-id="aa504-108">IActionResult</span><span class="sxs-lookup"><span data-stu-id="aa504-108">IActionResult</span></span>](#iactionresult-type)
::: moniker-end
::: moniker range=">= aspnetcore-2.1"
* [<span data-ttu-id="aa504-109">Tipo specifico</span><span class="sxs-lookup"><span data-stu-id="aa504-109">Specific type</span></span>](#specific-type)
* [<span data-ttu-id="aa504-110">IActionResult</span><span class="sxs-lookup"><span data-stu-id="aa504-110">IActionResult</span></span>](#iactionresult-type)
* [<span data-ttu-id="aa504-111">ActionResult\<T></span><span class="sxs-lookup"><span data-stu-id="aa504-111">ActionResult\<T></span></span>](#actionresultt-type)
::: moniker-end

<span data-ttu-id="aa504-112">Questo documento spiega quando è più appropriato utilizzare ogni tipo restituito.</span><span class="sxs-lookup"><span data-stu-id="aa504-112">This document explains when it's most appropriate to use each return type.</span></span>

## <a name="specific-type"></a><span data-ttu-id="aa504-113">Tipo specifico</span><span class="sxs-lookup"><span data-stu-id="aa504-113">Specific type</span></span>

<span data-ttu-id="aa504-114">L'azione più semplice restituisce un tipo di dati primitivo o complesso, ad esempio, `string` o un tipo di oggetto personalizzato.</span><span class="sxs-lookup"><span data-stu-id="aa504-114">The simplest action returns a primitive or complex data type (for example, `string` or a custom object type).</span></span> <span data-ttu-id="aa504-115">Si consideri l'azione seguente, che restituisce una raccolta di oggetti `Product` personalizzati:</span><span class="sxs-lookup"><span data-stu-id="aa504-115">Consider the following action, which returns a collection of custom `Product` objects:</span></span>

[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_Get)]

<span data-ttu-id="aa504-116">In assenza di condizioni note da cui proteggersi durante l'esecuzione dell'azione, la restituzione di un tipo specifico potrebbe essere sufficiente.</span><span class="sxs-lookup"><span data-stu-id="aa504-116">Without known conditions to safeguard against during action execution, returning a specific type could suffice.</span></span> <span data-ttu-id="aa504-117">L'azione precedente non accetta parametri, quindi non è necessario eseguire la convalida dei vincoli dei parametri.</span><span class="sxs-lookup"><span data-stu-id="aa504-117">The preceding action accepts no parameters, so parameter constraints validation isn't needed.</span></span>

<span data-ttu-id="aa504-118">Quando per un'azione devono essere prese in considerazione condizioni note, vengono introdotti più percorsi di ritorno.</span><span class="sxs-lookup"><span data-stu-id="aa504-118">When known conditions need to be accounted for in an action, multiple return paths are introduced.</span></span> <span data-ttu-id="aa504-119">In tal caso, è comune combinare un tipo restituito [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) al tipo restituito primitivo o complesso.</span><span class="sxs-lookup"><span data-stu-id="aa504-119">In such a case, it's common to mix an [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) return type with the primitive or complex return type.</span></span> <span data-ttu-id="aa504-120">Ai fini di questo tipo di azione, è necessario usare [IActionResult](#iactionresult-type) oppure [ActionResult\<T >](#actionresultt-type).</span><span class="sxs-lookup"><span data-stu-id="aa504-120">Either [IActionResult](#iactionresult-type) or [ActionResult\<T>](#actionresultt-type) are necessary to accommodate this type of action.</span></span>

## <a name="iactionresult-type"></a><span data-ttu-id="aa504-121">Tipo IActionResult</span><span class="sxs-lookup"><span data-stu-id="aa504-121">IActionResult type</span></span>

<span data-ttu-id="aa504-122">Il tipo restituito [IActionResult](/dotnet/api/microsoft.aspnetcore.mvc.iactionresult) è appropriato quando un'azione supporta più tipi restituiti [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult).</span><span class="sxs-lookup"><span data-stu-id="aa504-122">The [IActionResult](/dotnet/api/microsoft.aspnetcore.mvc.iactionresult) return type is appropriate when multiple [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) return types are possible in an action.</span></span> <span data-ttu-id="aa504-123">I tipi `ActionResult` rappresentano vari codici di stato HTTP.</span><span class="sxs-lookup"><span data-stu-id="aa504-123">The `ActionResult` types represent various HTTP status codes.</span></span> <span data-ttu-id="aa504-124">Alcuni tipi restituiti comuni che rientrano in questa categoria sono [BadRequestResult](/dotnet/api/microsoft.aspnetcore.mvc.badrequestresult) (400), [NotFoundResult](/dotnet/api/microsoft.aspnetcore.mvc.notfoundresult) (404) e [OkObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.okobjectresult) (200).</span><span class="sxs-lookup"><span data-stu-id="aa504-124">Some common return types falling into this category are [BadRequestResult](/dotnet/api/microsoft.aspnetcore.mvc.badrequestresult) (400), [NotFoundResult](/dotnet/api/microsoft.aspnetcore.mvc.notfoundresult) (404), and [OkObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.okobjectresult) (200).</span></span>

<span data-ttu-id="aa504-125">Dal momento che nell'azione sono presenti più percorsi e tipi restituiti, è necessario usare spesso l'attributo [[ProducesResponseType]](/dotnet/api/microsoft.aspnetcore.mvc.producesresponsetypeattribute.-ctor).</span><span class="sxs-lookup"><span data-stu-id="aa504-125">Because there are multiple return types and paths in the action, liberal use of the [[ProducesResponseType]](/dotnet/api/microsoft.aspnetcore.mvc.producesresponsetypeattribute.-ctor) attribute is necessary.</span></span> <span data-ttu-id="aa504-126">Questo attributo genera dettagli più descrittivi sulla risposta per le pagine della Guida di API generate da strumenti quali [Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).</span><span class="sxs-lookup"><span data-stu-id="aa504-126">This attribute produces more descriptive response details for API help pages generated by tools like [Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).</span></span> <span data-ttu-id="aa504-127">`[ProducesResponseType]` indica i tipi noti e i codici di stato HTTP che l'azione deve restituire.</span><span class="sxs-lookup"><span data-stu-id="aa504-127">`[ProducesResponseType]` indicates the known types and HTTP status codes to be returned by the action.</span></span>

### <a name="synchronous-action"></a><span data-ttu-id="aa504-128">Azione sincrona</span><span class="sxs-lookup"><span data-stu-id="aa504-128">Synchronous action</span></span>

<span data-ttu-id="aa504-129">Si consideri la seguente azione sincrona che prevede due possibili tipi restituiti:</span><span class="sxs-lookup"><span data-stu-id="aa504-129">Consider the following synchronous action in which there are two possible return types:</span></span>

[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.Pre21/Controllers/ProductsController.cs?name=snippet_GetById&highlight=8,11)]

<span data-ttu-id="aa504-130">Nell'azione precedente, viene restituito un codice di stato 404 quando il prodotto rappresentato da `id` non esiste nell'archivio dati sottostante.</span><span class="sxs-lookup"><span data-stu-id="aa504-130">In the preceding action, a 404 status code is returned when the product represented by `id` doesn't exist in the underlying data store.</span></span> <span data-ttu-id="aa504-131">Il metodo helper [NotFound](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.notfound) viene richiamato come collegamento a `return new NotFoundResult();`.</span><span class="sxs-lookup"><span data-stu-id="aa504-131">The [NotFound](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.notfound) helper method is invoked as a shortcut to `return new NotFoundResult();`.</span></span> <span data-ttu-id="aa504-132">Se il prodotto esiste, viene restituito un oggetto `Product` che rappresenta il payload con il codice di stato 200.</span><span class="sxs-lookup"><span data-stu-id="aa504-132">If the product does exist, a `Product` object representing the payload is returned with a 200 status code.</span></span> <span data-ttu-id="aa504-133">Il metodo helper [Ok](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.ok) viene richiamato come forma abbreviata di `return new OkObjectResult(product);`.</span><span class="sxs-lookup"><span data-stu-id="aa504-133">The [Ok](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.ok) helper method is invoked as the shorthand form of `return new OkObjectResult(product);`.</span></span>

### <a name="asynchronous-action"></a><span data-ttu-id="aa504-134">Azione asincrona</span><span class="sxs-lookup"><span data-stu-id="aa504-134">Asynchronous action</span></span>

<span data-ttu-id="aa504-135">Si consideri la seguente azione asincrona che prevede due possibili tipi restituiti:</span><span class="sxs-lookup"><span data-stu-id="aa504-135">Consider the following asynchronous action in which there are two possible return types:</span></span>

[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.Pre21/Controllers/ProductsController.cs?name=snippet_CreateAsync&highlight=8,13)]

<span data-ttu-id="aa504-136">Nell'azione precedente, viene restituito un codice di stato 400 quando si verifica un errore di convalida del modello e il metodo helper [BadRequest](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.badrequest) viene richiamato.</span><span class="sxs-lookup"><span data-stu-id="aa504-136">In the preceding action, a 400 status code is returned when model validation fails and the [BadRequest](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.badrequest) helper method is invoked.</span></span> <span data-ttu-id="aa504-137">Ad esempio, il modello seguente indica che le richieste devono fornire la proprietà `Name` e un valore.</span><span class="sxs-lookup"><span data-stu-id="aa504-137">For example, the following model indicates that requests must provide the `Name` property and a value.</span></span> <span data-ttu-id="aa504-138">Pertanto, se nella richiesta non viene fornito un `Name` corretto, la convalida del modello dà esito negativo.</span><span class="sxs-lookup"><span data-stu-id="aa504-138">Therefore, failure to provide a proper `Name` in the request causes model validation to fail.</span></span>

[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.DataAccess/Models/Product.cs?name=snippet_ProductClass&highlight=5-6)]

<span data-ttu-id="aa504-139">L'altro codice restituito conosciuto dall'azione precedente è 201, che viene generato dal metodo helper [CreatedAtAction](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdataction).</span><span class="sxs-lookup"><span data-stu-id="aa504-139">The preceding action's other known return code is a 201, which is generated by the [CreatedAtAction](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdataction) helper method.</span></span> <span data-ttu-id="aa504-140">In questo percorso, viene restituito l'oggetto `Product`.</span><span class="sxs-lookup"><span data-stu-id="aa504-140">In this path, the `Product` object is returned.</span></span>

::: moniker range=">= aspnetcore-2.1"
## <a name="actionresultt-type"></a><span data-ttu-id="aa504-141">Tipo ActionResult\<T></span><span class="sxs-lookup"><span data-stu-id="aa504-141">ActionResult\<T> type</span></span>

<span data-ttu-id="aa504-142">ASP.NET Core 2.1 introduce il tipo restituito [ActionResult\<T >](/dotnet/api/microsoft.aspnetcore.mvc.actionresult-1) per le azioni del controller API Web.</span><span class="sxs-lookup"><span data-stu-id="aa504-142">ASP.NET Core 2.1 introduces the [ActionResult\<T>](/dotnet/api/microsoft.aspnetcore.mvc.actionresult-1) return type for Web API controller actions.</span></span> <span data-ttu-id="aa504-143">Consente di restituire un tipo che deriva da [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) o di restituire un [tipo specifico](#specific-type).</span><span class="sxs-lookup"><span data-stu-id="aa504-143">It enables you to return a type deriving from [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) or return a [specific type](#specific-type).</span></span> <span data-ttu-id="aa504-144">`ActionResult<T>` offre i vantaggi seguenti rispetto al [tipo IActionResult](#iactionresult-type):</span><span class="sxs-lookup"><span data-stu-id="aa504-144">`ActionResult<T>` offers the following benefits over the [IActionResult type](#iactionresult-type):</span></span>

* <span data-ttu-id="aa504-145">La proprietà `Type` dell'attributo [[ProducesResponseType]](/dotnet/api/microsoft.aspnetcore.mvc.producesresponsetypeattribute) può essere esclusa.</span><span class="sxs-lookup"><span data-stu-id="aa504-145">The [[ProducesResponseType]](/dotnet/api/microsoft.aspnetcore.mvc.producesresponsetypeattribute) attribute's `Type` property can be excluded.</span></span>
* <span data-ttu-id="aa504-146">[Operatori di cast impliciti](/dotnet/csharp/language-reference/keywords/implicit) supportano la conversione di `T` e `ActionResult` in `ActionResult<T>`.</span><span class="sxs-lookup"><span data-stu-id="aa504-146">[Implicit cast operators](/dotnet/csharp/language-reference/keywords/implicit) support the conversion of both `T` and `ActionResult` to `ActionResult<T>`.</span></span> <span data-ttu-id="aa504-147">`T` viene convertito in [ObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.objectresult), il che significa che `return new ObjectResult(T);` viene semplificato in `return T;`.</span><span class="sxs-lookup"><span data-stu-id="aa504-147">`T` converts to [ObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.objectresult), which means `return new ObjectResult(T);` is simplified to `return T;`.</span></span>

<span data-ttu-id="aa504-148">La maggior parte delle azioni presenta un tipo restituito specifico.</span><span class="sxs-lookup"><span data-stu-id="aa504-148">Most actions have a specific return type.</span></span> <span data-ttu-id="aa504-149">Durante l'esecuzione di un'azione possono verificarsi condizioni impreviste, nel qual caso il tipo specifico non viene restituito.</span><span class="sxs-lookup"><span data-stu-id="aa504-149">Unexpected conditions can occur during action execution, in which case the specific type isn't returned.</span></span> <span data-ttu-id="aa504-150">Ad esempio, il parametro di input di un'azione potrebbe non superare la convalida del modello.</span><span class="sxs-lookup"><span data-stu-id="aa504-150">For example, an action's input parameter may fail model validation.</span></span> <span data-ttu-id="aa504-151">In tal caso, è consueto che venga restituito il tipo `ActionResult` appropriato anziché il tipo specifico.</span><span class="sxs-lookup"><span data-stu-id="aa504-151">In such a case, it's common to return the appropriate `ActionResult` type instead of the specific type.</span></span>

### <a name="synchronous-action"></a><span data-ttu-id="aa504-152">Azione sincrona</span><span class="sxs-lookup"><span data-stu-id="aa504-152">Synchronous action</span></span>

<span data-ttu-id="aa504-153">Si consideri un'azione sincrona che prevede due possibili tipi restituiti:</span><span class="sxs-lookup"><span data-stu-id="aa504-153">Consider a synchronous action in which there are two possible return types:</span></span>

<span data-ttu-id="aa504-154">[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_GetById&highlight=8,11)]</span><span class="sxs-lookup"><span data-stu-id="aa504-154">[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_GetById&highlight=8,11)]</span></span>

<span data-ttu-id="aa504-155">Nel codice precedente, viene restituito un codice di stato 404 quando il prodotto non esiste nel database.</span><span class="sxs-lookup"><span data-stu-id="aa504-155">In the preceding code, a 404 status code is returned when the product doesn't exist in the database.</span></span> <span data-ttu-id="aa504-156">Se il prodotto esiste, viene restituito l'oggetto `Product` corrispondente.</span><span class="sxs-lookup"><span data-stu-id="aa504-156">If the product does exist, the corresponding `Product` object is returned.</span></span> <span data-ttu-id="aa504-157">Prima di ASP.NET Core 2.1, la riga `return product;` sarebbe stata `return Ok(product);`.</span><span class="sxs-lookup"><span data-stu-id="aa504-157">Before ASP.NET Core 2.1, the `return product;` line would have been `return Ok(product);`.</span></span>

> [!TIP]
> <span data-ttu-id="aa504-158">A partire da ASP.NET Core 2.1, è attiva l'inferenza per l'origine di associazione del parametro di azione quando una classe controller è decorata con l'attributo`[ApiController]`.</span><span class="sxs-lookup"><span data-stu-id="aa504-158">As of ASP.NET Core 2.1, action parameter binding source inference is enabled when a controller class is decorated with the `[ApiController]` attribute.</span></span> <span data-ttu-id="aa504-159">Un nome di parametro corrispondente a un nome del modello di route viene associato automaticamente usando i dati della route della richiesta.</span><span class="sxs-lookup"><span data-stu-id="aa504-159">A parameter name matching a name in the route template is automatically bound using the request route data.</span></span> <span data-ttu-id="aa504-160">Di conseguenza, il parametro `id` dell'azione precedente non è annotato in modo esplicito con l'attributo [[FromRoute]](/dotnet/api/microsoft.aspnetcore.mvc.fromrouteattribute).</span><span class="sxs-lookup"><span data-stu-id="aa504-160">Consequently, the preceding action's `id` parameter isn't explicitly annotated with the [[FromRoute]](/dotnet/api/microsoft.aspnetcore.mvc.fromrouteattribute) attribute.</span></span>

### <a name="asynchronous-action"></a><span data-ttu-id="aa504-161">Azione asincrona</span><span class="sxs-lookup"><span data-stu-id="aa504-161">Asynchronous action</span></span>

<span data-ttu-id="aa504-162">Si consideri un'azione asincrona che prevede due possibili tipi restituiti:</span><span class="sxs-lookup"><span data-stu-id="aa504-162">Consider an asynchronous action in which there are two possible return types:</span></span>

<span data-ttu-id="aa504-163">[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_CreateAsync&highlight=8,13)]</span><span class="sxs-lookup"><span data-stu-id="aa504-163">[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_CreateAsync&highlight=8,13)]</span></span>

<span data-ttu-id="aa504-164">Se si verifica un errore di convalida del modello, viene richiamato il metodo [BadRequest](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.badrequest#Microsoft_AspNetCore_Mvc_ControllerBase_BadRequest_Microsoft_AspNetCore_Mvc_ModelBinding_ModelStateDictionary_) per restituire un codice di stato 400.</span><span class="sxs-lookup"><span data-stu-id="aa504-164">If model validation fails, the [BadRequest](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.badrequest#Microsoft_AspNetCore_Mvc_ControllerBase_BadRequest_Microsoft_AspNetCore_Mvc_ModelBinding_ModelStateDictionary_) method is invoked to return a 400 status code.</span></span> <span data-ttu-id="aa504-165">La proprietà [ModelState](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.modelstate) che contiene gli errori di convalida specifici viene passata al codice.</span><span class="sxs-lookup"><span data-stu-id="aa504-165">The [ModelState](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.modelstate) property containing the specific validation errors is passed to it.</span></span> <span data-ttu-id="aa504-166">Se la convalida del modello ha esito positivo, il prodotto viene creato nel database.</span><span class="sxs-lookup"><span data-stu-id="aa504-166">If model validation succeeds, the product is created in the database.</span></span> <span data-ttu-id="aa504-167">Viene restituito il codice di stato 201.</span><span class="sxs-lookup"><span data-stu-id="aa504-167">A 201 status code is returned.</span></span>

> [!TIP]
> <span data-ttu-id="aa504-168">A partire da ASP.NET Core 2.1, è attiva l'inferenza per l'origine di associazione del parametro di azione quando una classe controller è decorata con l'attributo`[ApiController]`.</span><span class="sxs-lookup"><span data-stu-id="aa504-168">As of ASP.NET Core 2.1, action parameter binding source inference is enabled when a controller class is decorated with the `[ApiController]` attribute.</span></span> <span data-ttu-id="aa504-169">I parametri di tipo complesso vengono associati automaticamente tramite il corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="aa504-169">Complex type parameters are automatically bound using the request body.</span></span> <span data-ttu-id="aa504-170">Di conseguenza, il parametro `product` dell'azione precedente non è annotato in modo esplicito con l'attributo [[FromBody]](/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute).</span><span class="sxs-lookup"><span data-stu-id="aa504-170">Consequently, the preceding action's `product` parameter isn't explicitly annotated with the [[FromBody]](/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute) attribute.</span></span>
::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="aa504-171">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="aa504-171">Additional resources</span></span>

* [<span data-ttu-id="aa504-172">Azioni del controller</span><span class="sxs-lookup"><span data-stu-id="aa504-172">Controller actions</span></span>](xref:mvc/controllers/actions)
* [<span data-ttu-id="aa504-173">Convalida modello</span><span class="sxs-lookup"><span data-stu-id="aa504-173">Model validation</span></span>](xref:mvc/models/validation)
* [<span data-ttu-id="aa504-174">Pagine della Guida dell'API Web con Swagger</span><span class="sxs-lookup"><span data-stu-id="aa504-174">Web API help pages using Swagger</span></span>](xref:tutorials/web-api-help-pages-using-swagger)