---
title: Веб-пакет SDK для ASP.NET Core
author: Rick-Anderson
description: Общие сведения о пакете Microsoft.NET.Sdk.Web.
ms.author: riande
ms.date: 01/25/2020
no-loc:
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: razor-pages/web-sdk
ms.openlocfilehash: 163bc2679deda449f97cb4e50da1093e6b1edda4
ms.sourcegitcommit: 65add17f74a29a647d812b04517e46cbc78258f9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/19/2020
ms.locfileid: "88634817"
---
# <a name="aspnet-core-web-sdk"></a><span data-ttu-id="5a8d0-103">Веб-пакет SDK для ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="5a8d0-103">ASP.NET Core Web SDK</span></span>

### <a name="overview"></a><span data-ttu-id="5a8d0-104">Обзор</span><span class="sxs-lookup"><span data-stu-id="5a8d0-104">Overview</span></span>

<span data-ttu-id="5a8d0-105">`Microsoft.NET.Sdk.Web` — это [пакет SDK проекта MSBuild](https://docs.microsoft.com/visualstudio/msbuild/how-to-use-project-sdk) для разработки приложений ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="5a8d0-105">`Microsoft.NET.Sdk.Web` is an [MSBuild project SDK](https://docs.microsoft.com/visualstudio/msbuild/how-to-use-project-sdk) for building ASP.NET Core apps.</span></span> <span data-ttu-id="5a8d0-106">Приложения ASP.NET Core можно создавать и без этого пакета SDK, однако веб-пакет SDK дает следующие преимущества:</span><span class="sxs-lookup"><span data-stu-id="5a8d0-106">It's possible to build an ASP.NET Core app without this SDK, however, the Web SDK is:</span></span>

* <span data-ttu-id="5a8d0-107">обеспечивает максимальное удобство разработки;</span><span class="sxs-lookup"><span data-stu-id="5a8d0-107">Tailored towards providing a first-class experience.</span></span>
* <span data-ttu-id="5a8d0-108">рекомендуется для большинства пользователей.</span><span class="sxs-lookup"><span data-stu-id="5a8d0-108">The recommended target for most users.</span></span>

<span data-ttu-id="5a8d0-109">Используйте Web.SDK в проекте:</span><span class="sxs-lookup"><span data-stu-id="5a8d0-109">Use the Web.SDK in a project:</span></span>

  ```xml
  <Project Sdk="Microsoft.NET.Sdk.Web">
    <!-- omitted for brevity -->
  </Project>
  ```

<span data-ttu-id="5a8d0-110">Возможности, обеспечиваемые веб-пакетом SDK:</span><span class="sxs-lookup"><span data-stu-id="5a8d0-110">Features enabled by using the Web SDK:</span></span>

* <span data-ttu-id="5a8d0-111">Проекты, предназначенные для .NET Core 3.0 или более поздней версии, имеют неявные ссылки на следующие компоненты:</span><span class="sxs-lookup"><span data-stu-id="5a8d0-111">Projects targeting .NET Core 3.0 or later implicitly reference:</span></span>

  * <span data-ttu-id="5a8d0-112">[общая платформа ASP.NET Core](xref:fundamentals/metapackage-app);</span><span class="sxs-lookup"><span data-stu-id="5a8d0-112">The [ASP.NET Core shared framework](xref:fundamentals/metapackage-app).</span></span>
  * <span data-ttu-id="5a8d0-113">[анализаторы](/visualstudio/extensibility/getting-started-with-roslyn-analyzers), предназначенные для создания приложений ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="5a8d0-113">[Analyzers](/visualstudio/extensibility/getting-started-with-roslyn-analyzers) designed for building ASP.NET Core apps.</span></span>
* <span data-ttu-id="5a8d0-114">Веб-пакет SDK импортирует целевые объекты MSBuild, которые позволяют использовать профили публикации и выполнять публикацию с помощью WebDeploy.</span><span class="sxs-lookup"><span data-stu-id="5a8d0-114">The Web SDK imports MSBuild targets that enable the use of publish profiles and publishing using WebDeploy.</span></span>

### <a name="properties"></a><span data-ttu-id="5a8d0-115">Свойства</span><span class="sxs-lookup"><span data-stu-id="5a8d0-115">Properties</span></span>

| <span data-ttu-id="5a8d0-116">Свойство.</span><span class="sxs-lookup"><span data-stu-id="5a8d0-116">Property</span></span> | <span data-ttu-id="5a8d0-117">Описание</span><span class="sxs-lookup"><span data-stu-id="5a8d0-117">Description</span></span> |
| -------- | ----------- |
| `DisableImplicitFrameworkReferences` | <span data-ttu-id="5a8d0-118">Отключает неявную ссылку на общую платформу `Microsoft.AspNetCore.App`.</span><span class="sxs-lookup"><span data-stu-id="5a8d0-118">Disables implicit reference to the `Microsoft.AspNetCore.App` shared framework.</span></span> |
| `DisableImplicitAspNetCoreAnalyzers` | <span data-ttu-id="5a8d0-119">Отключает неявную ссылку на анализаторы ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="5a8d0-119">Disables implicit reference to ASP.NET Core analyzers.</span></span> |
| `DisableImplicitComponentsAnalyzers` | <span data-ttu-id="5a8d0-120">Отключает неявную ссылку на анализаторы компонентов Razor при сборке приложений Blazor (серверных).</span><span class="sxs-lookup"><span data-stu-id="5a8d0-120">Disables implicit reference to Razor Components analyzers when building Blazor (server) applications.</span></span> |
