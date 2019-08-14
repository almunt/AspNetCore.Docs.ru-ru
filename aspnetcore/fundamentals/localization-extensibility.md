---
title: Расширяемость локализации
author: hishamco
description: Узнайте, как расширить интерфейсы API локализации в приложениях ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/03/2019
uid: fundamentals/localization-extensibility
ms.openlocfilehash: 92fe954ea6bf5d0a8f9f62f4da696d197c51af04
ms.sourcegitcommit: 4fe3ae892f54dc540859bff78741a28c2daa9a38
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/04/2019
ms.locfileid: "68776759"
---
# <a name="localization-extensibility"></a><span data-ttu-id="3a828-103">Расширяемость локализации</span><span class="sxs-lookup"><span data-stu-id="3a828-103">Localization Extensibility</span></span>

<span data-ttu-id="3a828-104">Автор: [Хишам Бин Атея](https://github.com/hishamco) (Hisham Bin Ateya)</span><span class="sxs-lookup"><span data-stu-id="3a828-104">By [Hisham Bin Ateya](https://github.com/hishamco)</span></span>

<span data-ttu-id="3a828-105">В этой статье:</span><span class="sxs-lookup"><span data-stu-id="3a828-105">This article:</span></span>

* <span data-ttu-id="3a828-106">Перечисляет точки расширения в API локализации.</span><span class="sxs-lookup"><span data-stu-id="3a828-106">Lists the extensibility points on the localization APIs.</span></span>
* <span data-ttu-id="3a828-107">В этой статье приводятся инструкции по расширениям локализации в приложениях ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="3a828-107">Provides instructions on how to extend ASP.NET Core app localization.</span></span>

## <a name="extensible-points-in-localization-apis"></a><span data-ttu-id="3a828-108">Точки расширения в API локализации</span><span class="sxs-lookup"><span data-stu-id="3a828-108">Extensible Points in Localization APIs</span></span>

<span data-ttu-id="3a828-109">API локализации ASP.NET Core созданы с учетом потребностей расширения.</span><span class="sxs-lookup"><span data-stu-id="3a828-109">ASP.NET Core localization APIs are built to be extensible.</span></span> <span data-ttu-id="3a828-110">Расширяемость позволяет разработчикам настраивать локализацию в соответствии со своими потребностями.</span><span class="sxs-lookup"><span data-stu-id="3a828-110">Extensibility allows developers to customize the localization according to their needs.</span></span> <span data-ttu-id="3a828-111">Например, [OrchardCore](https://github.com/orchardCMS/OrchardCore/) имеет `POStringLocalizer`.</span><span class="sxs-lookup"><span data-stu-id="3a828-111">For instance, [OrchardCore](https://github.com/orchardCMS/OrchardCore/) has a `POStringLocalizer`.</span></span> <span data-ttu-id="3a828-112">В `POStringLocalizer` приводится подробное описание использования [локализации переносимых объектов](xref:fundamentals/portable-object-localization) для использования файлов `PO` для хранения ресурсов локализации.</span><span class="sxs-lookup"><span data-stu-id="3a828-112">`POStringLocalizer` describes in detail using [Portable Object localization](xref:fundamentals/portable-object-localization) to use `PO` files to store localization resources.</span></span>

<span data-ttu-id="3a828-113">В этой статье описаны две основные точки расширения, предоставляемые API локализации.</span><span class="sxs-lookup"><span data-stu-id="3a828-113">This article lists the two main extensibility points that localization APIs provide:</span></span> 

* <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider>
* <xref:Microsoft.Extensions.Localization.IStringLocalizer>

## <a name="localization-culture-providers"></a><span data-ttu-id="3a828-114">Поставщики языка и региональных параметров локализации</span><span class="sxs-lookup"><span data-stu-id="3a828-114">Localization Culture Providers</span></span>

<span data-ttu-id="3a828-115">API локализации ASP.NET Core имеют четыре поставщика по умолчанию, которые могут определить текущий язык и региональные параметры выполняемого запроса:</span><span class="sxs-lookup"><span data-stu-id="3a828-115">ASP.NET Core localization APIs have four default providers that can determine the current culture of an executing request:</span></span>

* <xref:Microsoft.AspNetCore.Localization.QueryStringRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.CookieRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.AcceptLanguageHeaderRequestCultureProvider>
* <xref:Microsoft.AspNetCore.Localization.CustomRequestCultureProvider>

<span data-ttu-id="3a828-116">Указанные выше поставщики подробно описаны в документации [ПО промежуточного слоя локализации](xref:fundamentals/localization).</span><span class="sxs-lookup"><span data-stu-id="3a828-116">The preceding providers are described in detail in the [Localization Middleware](xref:fundamentals/localization) documentation.</span></span> <span data-ttu-id="3a828-117">Если поставщики по умолчанию не отвечают вашим потребностям, создайте свой поставщик, используя один из следующих подходов:</span><span class="sxs-lookup"><span data-stu-id="3a828-117">If the default providers don't meet your needs, build a custom provider using one of the following approaches:</span></span>

### <a name="use-customrequestcultureprovider"></a><span data-ttu-id="3a828-118">Используя CustomRequestCultureProvider</span><span class="sxs-lookup"><span data-stu-id="3a828-118">Use CustomRequestCultureProvider</span></span>

<span data-ttu-id="3a828-119"><xref:Microsoft.AspNetCore.Localization.CustomRequestCultureProvider> предоставляет пользовательский <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider>, использующий простой делегат для определения текущего языка и региональных параметров локализации:</span><span class="sxs-lookup"><span data-stu-id="3a828-119"><xref:Microsoft.AspNetCore.Localization.CustomRequestCultureProvider> provides a custom <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> that uses a simple delegate to determine the current localization culture:</span></span>

::: moniker range=">= aspnetcore-2.2"

```csharp
options.AddInitialRequestCultureProvider(
    new CustomRequestCultureProvider(async context =>
{
    var currentCulture = "en";
    var segments = context.Request.Path.Value.Split(new char[] { '/' }, 
        StringSplitOptions.RemoveEmptyEntries);

    if (segments.Length > 1 && segments[0].Length == 2)
    {
        currentCulture = segments[0];
    }

    var requestCulture = new ProviderCultureResult(culture);
    
    return Task.FromResult(requestCulture);
}));
```

::: moniker-end

::: moniker range="< aspnetcore-2.2"

```csharp
options.RequestCultureProviders.Insert(0, 
    new CustomRequestCultureProvider(async context =>
{
    var currentCulture = "en";
    var segments = context.Request.Path.Value.Split(new char[] { '/' }, 
        StringSplitOptions.RemoveEmptyEntries);

    if (segments.Length > 1 && segments[0].Length == 2)
    {
        currentCulture = segments[0];
    }

    var requestCulture = new ProviderCultureResult(culture);
    
    return Task.FromResult(requestCulture);
}));
```

::: moniker-end

### <a name="use-a-new-implemetation-of-requestcultureprovider"></a><span data-ttu-id="3a828-120">Используя новую реализацию RequestCultureProvider</span><span class="sxs-lookup"><span data-stu-id="3a828-120">Use a new implemetation of RequestCultureProvider</span></span>

<span data-ttu-id="3a828-121">Можно создать новую реализацию <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider>, которая определяет сведения о языке и региональных параметрах запроса из пользовательского источника.</span><span class="sxs-lookup"><span data-stu-id="3a828-121">A new implementation of <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> can be created that determines the request culture information from a custom source.</span></span> <span data-ttu-id="3a828-122">Например, пользовательский источник может быть файлом конфигурации или базой данных.</span><span class="sxs-lookup"><span data-stu-id="3a828-122">For example, the custom source can be a configuration file or database.</span></span>

<span data-ttu-id="3a828-123">В следующем примере показан `AppSettingsRequestCultureProvider`, который расширяет класс <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> для определения сведений о языке и региональных параметрах запроса из *appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="3a828-123">The following example shows `AppSettingsRequestCultureProvider`, which extends the <xref:Microsoft.AspNetCore.Localization.RequestCultureProvider> to determine the request culture information from *appsettings.json*:</span></span>

```csharp
public class AppSettingsRequestCultureProvider : RequestCultureProvider
{
    public string CultureKey { get; set; } = "culture";

    public string UICultureKey { get; set; } = "ui-culture";

    public override Task<ProviderCultureResult> DetermineProviderCultureResult(HttpContext httpContext)
    {
        if (httpContext == null)
        {
            throw new ArgumentNullException();
        }

        var configuration = httpContext.RequestServices.GetService<IConfigurationRoot>();
        var culture = configuration[CultureKey];
        var uiCulture = configuration[UICultureKey];

        if (culture == null && uiCulture == null)
        {
            return Task.FromResult((ProviderCultureResult)null);
        }

        if (culture != null && uiCulture == null)
        {
            uiCulture = culture;
        }

        if (culture == null && uiCulture != null)
        {
            culture = uiCulture;
        }
        
        var providerResultCulture = new ProviderCultureResult(culture, uiCulture);

        return Task.FromResult(providerResultCulture);
    }
}
```

## <a name="localization-resources"></a><span data-ttu-id="3a828-124">Ресурсы локализации</span><span class="sxs-lookup"><span data-stu-id="3a828-124">Localization resources</span></span>

<span data-ttu-id="3a828-125">Для локализации в ASP.NET Core есть <xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer>.</span><span class="sxs-lookup"><span data-stu-id="3a828-125">ASP.NET Core localization provides <xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer>.</span></span> <span data-ttu-id="3a828-126"><xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer> — это реализация <xref:Microsoft.Extensions.Localization.IStringLocalizer>, которая использует `resx` для хранения ресурсов локализации.</span><span class="sxs-lookup"><span data-stu-id="3a828-126"><xref:Microsoft.Extensions.Localization.ResourceManagerStringLocalizer> is an implementation of <xref:Microsoft.Extensions.Localization.IStringLocalizer> that is uses `resx` to store localization resources.</span></span>

<span data-ttu-id="3a828-127">Вы не ограничены использованием файлов `resx`.</span><span class="sxs-lookup"><span data-stu-id="3a828-127">You aren't limited to using `resx` files.</span></span> <span data-ttu-id="3a828-128">При реализации `IStringLocalized` можно использовать любой источник данных.</span><span class="sxs-lookup"><span data-stu-id="3a828-128">By implementing `IStringLocalized`, any data source can be used.</span></span>

<span data-ttu-id="3a828-129">В следующих примерах проектов реализуется <xref:Microsoft.Extensions.Localization.IStringLocalizer>:</span><span class="sxs-lookup"><span data-stu-id="3a828-129">The following example projects implement <xref:Microsoft.Extensions.Localization.IStringLocalizer>:</span></span> 

* [<span data-ttu-id="3a828-130">EFStringLocalizer</span><span class="sxs-lookup"><span data-stu-id="3a828-130">EFStringLocalizer</span></span>](https://github.com/aspnet/Entropy/tree/master/samples/Localization.EntityFramework)
* [<span data-ttu-id="3a828-131">JsonStringLocalizer</span><span class="sxs-lookup"><span data-stu-id="3a828-131">JsonStringLocalizer</span></span>](https://github.com/hishamco/My.Extensions.Localization.Json)
* [<span data-ttu-id="3a828-132">SqlLocalizer</span><span class="sxs-lookup"><span data-stu-id="3a828-132">SqlLocalizer</span></span>](https://github.com/damienbod/AspNetCoreLocalization)