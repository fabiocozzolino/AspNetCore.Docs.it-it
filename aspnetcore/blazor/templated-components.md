---
title: ASP.NET Core Blazor componenti basati su modelli
author: guardrex
description: Informazioni su come i componenti basati su modelli possono accettare uno o più modelli di interfaccia utente come parametri, che possono quindi essere usati come parte della logica di rendering del componente.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/12/2020
no-loc:
- Blazor
- SignalR
uid: blazor/templated-components
ms.openlocfilehash: b64d6a731e540b13c50b2c6108f75efd0ac9290c
ms.sourcegitcommit: 9a129f5f3e31cc449742b164d5004894bfca90aa
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/06/2020
ms.locfileid: "78657717"
---
# <a name="aspnet-core-opno-locblazor-templated-components"></a><span data-ttu-id="64309-103">ASP.NET Core Blazor componenti basati su modelli</span><span class="sxs-lookup"><span data-stu-id="64309-103">ASP.NET Core Blazor templated components</span></span>

<span data-ttu-id="64309-104">Di [Luke Latham](https://github.com/guardrex) e [Daniel Roth](https://github.com/danroth27)</span><span class="sxs-lookup"><span data-stu-id="64309-104">By [Luke Latham](https://github.com/guardrex) and [Daniel Roth](https://github.com/danroth27)</span></span>

<span data-ttu-id="64309-105">I componenti basati su modelli sono componenti che accettano uno o più modelli di interfaccia utente come parametri, che possono quindi essere usati come parte della logica di rendering del componente.</span><span class="sxs-lookup"><span data-stu-id="64309-105">Templated components are components that accept one or more UI templates as parameters, which can then be used as part of the component's rendering logic.</span></span> <span data-ttu-id="64309-106">I componenti basati su modelli consentono di creare componenti di livello superiore più riutilizzabili rispetto ai componenti normali.</span><span class="sxs-lookup"><span data-stu-id="64309-106">Templated components allow you to author higher-level components that are more reusable than regular components.</span></span> <span data-ttu-id="64309-107">Di seguito sono riportati alcuni esempi:</span><span class="sxs-lookup"><span data-stu-id="64309-107">A couple of examples include:</span></span>

* <span data-ttu-id="64309-108">Componente della tabella che consente a un utente di specificare i modelli per l'intestazione, le righe e il piè di pagina della tabella.</span><span class="sxs-lookup"><span data-stu-id="64309-108">A table component that allows a user to specify templates for the table's header, rows, and footer.</span></span>
* <span data-ttu-id="64309-109">Componente di elenco che consente a un utente di specificare un modello per il rendering degli elementi in un elenco.</span><span class="sxs-lookup"><span data-stu-id="64309-109">A list component that allows a user to specify a template for rendering items in a list.</span></span>

## <a name="template-parameters"></a><span data-ttu-id="64309-110">Parametri del modello</span><span class="sxs-lookup"><span data-stu-id="64309-110">Template parameters</span></span>

<span data-ttu-id="64309-111">Un componente basato su modelli viene definito specificando uno o più parametri del componente di tipo `RenderFragment` o `RenderFragment<T>`.</span><span class="sxs-lookup"><span data-stu-id="64309-111">A templated component is defined by specifying one or more component parameters of type `RenderFragment` or `RenderFragment<T>`.</span></span> <span data-ttu-id="64309-112">Un frammento di rendering rappresenta un segmento di interfaccia utente di cui eseguire il rendering.</span><span class="sxs-lookup"><span data-stu-id="64309-112">A render fragment represents a segment of UI to render.</span></span> <span data-ttu-id="64309-113">`RenderFragment<T>` accetta un parametro di tipo che può essere specificato quando viene richiamato il frammento di rendering.</span><span class="sxs-lookup"><span data-stu-id="64309-113">`RenderFragment<T>` takes a type parameter that can be specified when the render fragment is invoked.</span></span>

<span data-ttu-id="64309-114">componente `TableTemplate`:</span><span class="sxs-lookup"><span data-stu-id="64309-114">`TableTemplate` component:</span></span>

[!code-razor[](common/samples/3.x/BlazorWebAssemblySample/Components/TableTemplate.razor)]

<span data-ttu-id="64309-115">Quando si usa un componente basato su modelli, è possibile specificare i parametri del modello usando gli elementi figlio che corrispondono ai nomi dei parametri (`TableHeader` e `RowTemplate` nell'esempio seguente):</span><span class="sxs-lookup"><span data-stu-id="64309-115">When using a templated component, the template parameters can be specified using child elements that match the names of the parameters (`TableHeader` and `RowTemplate` in the following example):</span></span>

```razor
<TableTemplate Items="pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@context.PetId</td>
        <td>@context.Name</td>
    </RowTemplate>
</TableTemplate>
```

## <a name="template-context-parameters"></a><span data-ttu-id="64309-116">Parametri di contesto del modello</span><span class="sxs-lookup"><span data-stu-id="64309-116">Template context parameters</span></span>

<span data-ttu-id="64309-117">Gli argomenti del componente di tipo `RenderFragment<T>` passati come elementi hanno un parametro implicito denominato `context`, ad esempio dall'esempio di codice precedente, `@context.PetId`, ma è possibile modificare il nome del parametro usando l'attributo `Context` nell'elemento figlio.</span><span class="sxs-lookup"><span data-stu-id="64309-117">Component arguments of type `RenderFragment<T>` passed as elements have an implicit parameter named `context` (for example from the preceding code sample, `@context.PetId`), but you can change the parameter name using the `Context` attribute on the child element.</span></span> <span data-ttu-id="64309-118">Nell'esempio seguente, l'attributo `Context` dell'elemento `RowTemplate` specifica il parametro `pet`:</span><span class="sxs-lookup"><span data-stu-id="64309-118">In the following example, the `RowTemplate` element's `Context` attribute specifies the `pet` parameter:</span></span>

```razor
<TableTemplate Items="pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate Context="pet">
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>
```

<span data-ttu-id="64309-119">In alternativa, è possibile specificare l'attributo `Context` sull'elemento Component.</span><span class="sxs-lookup"><span data-stu-id="64309-119">Alternatively, you can specify the `Context` attribute on the component element.</span></span> <span data-ttu-id="64309-120">L'attributo `Context` specificato si applica a tutti i parametri di modello specificati.</span><span class="sxs-lookup"><span data-stu-id="64309-120">The specified `Context` attribute applies to all specified template parameters.</span></span> <span data-ttu-id="64309-121">Questa operazione può essere utile quando si desidera specificare il nome del parametro di contenuto per il contenuto figlio implicito (senza alcun elemento figlio di wrapping).</span><span class="sxs-lookup"><span data-stu-id="64309-121">This can be useful when you want to specify the content parameter name for implicit child content (without any wrapping child element).</span></span> <span data-ttu-id="64309-122">Nell'esempio seguente l'attributo `Context` viene visualizzato nell'elemento `TableTemplate` e si applica a tutti i parametri del modello:</span><span class="sxs-lookup"><span data-stu-id="64309-122">In the following example, the `Context` attribute appears on the `TableTemplate` element and applies to all template parameters:</span></span>

```razor
<TableTemplate Items="pets" Context="pet">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>
```

## <a name="generic-typed-components"></a><span data-ttu-id="64309-123">Componenti tipizzati in modo generico</span><span class="sxs-lookup"><span data-stu-id="64309-123">Generic-typed components</span></span>

<span data-ttu-id="64309-124">I componenti basati su modelli spesso sono tipizzati in modo generico.</span><span class="sxs-lookup"><span data-stu-id="64309-124">Templated components are often generically typed.</span></span> <span data-ttu-id="64309-125">È ad esempio possibile utilizzare un componente `ListViewTemplate` generico per eseguire il rendering dei valori `IEnumerable<T>`.</span><span class="sxs-lookup"><span data-stu-id="64309-125">For example, a generic `ListViewTemplate` component can be used to render `IEnumerable<T>` values.</span></span> <span data-ttu-id="64309-126">Per definire un componente generico, usare la direttiva [`@typeparam`](xref:mvc/views/razor#typeparam) per specificare i parametri di tipo:</span><span class="sxs-lookup"><span data-stu-id="64309-126">To define a generic component, use the [`@typeparam`](xref:mvc/views/razor#typeparam) directive to specify type parameters:</span></span>

[!code-razor[](common/samples/3.x/BlazorWebAssemblySample/Components/ListViewTemplate.razor)]

<span data-ttu-id="64309-127">Quando si usano componenti tipizzati generici, il parametro di tipo viene dedotto, se possibile:</span><span class="sxs-lookup"><span data-stu-id="64309-127">When using generic-typed components, the type parameter is inferred if possible:</span></span>

```razor
<ListViewTemplate Items="pets">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```

<span data-ttu-id="64309-128">In caso contrario, il parametro di tipo deve essere specificato in modo esplicito utilizzando un attributo che corrisponde al nome del parametro di tipo.</span><span class="sxs-lookup"><span data-stu-id="64309-128">Otherwise, the type parameter must be explicitly specified using an attribute that matches the name of the type parameter.</span></span> <span data-ttu-id="64309-129">Nell'esempio seguente `TItem="Pet"` specifica il tipo:</span><span class="sxs-lookup"><span data-stu-id="64309-129">In the following example, `TItem="Pet"` specifies the type:</span></span>

```razor
<ListViewTemplate Items="pets" TItem="Pet">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```