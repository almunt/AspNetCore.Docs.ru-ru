---
title: ASP.NET Core маршрутизация Блазор
author: guardrex
description: Узнайте, как маршрутизировать запросы в приложениях и о компоненте Навлинк.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 07/02/2019
uid: blazor/routing
ms.openlocfilehash: 70cae6b3a21fe3537d6841a6716398a5fc45db62
ms.sourcegitcommit: f30b18442ed12831c7e86b0db249183ccd749f59
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/23/2019
ms.locfileid: "68948314"
---
# <a name="aspnet-core-blazor-routing"></a><span data-ttu-id="b3790-103">ASP.NET Core маршрутизация Блазор</span><span class="sxs-lookup"><span data-stu-id="b3790-103">ASP.NET Core Blazor routing</span></span>

<span data-ttu-id="b3790-104">Автор [Люк Латэм](https://github.com/guardrex) (Luke Latham)</span><span class="sxs-lookup"><span data-stu-id="b3790-104">By [Luke Latham](https://github.com/guardrex)</span></span>

<span data-ttu-id="b3790-105">Узнайте, как маршрутизировать запросы и как использовать `NavLink` компонент для создания навигационных ссылок в приложениях блазор.</span><span class="sxs-lookup"><span data-stu-id="b3790-105">Learn how to route requests and how to use the `NavLink` component to create navigation links in Blazor apps.</span></span>

## <a name="aspnet-core-endpoint-routing-integration"></a><span data-ttu-id="b3790-106">Интеграция маршрутизации конечных точек ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="b3790-106">ASP.NET Core endpoint routing integration</span></span>

<span data-ttu-id="b3790-107">Серверная часть блазор интегрирована в [ASP.NET Coreную маршрутизацию конечных точек](xref:fundamentals/routing).</span><span class="sxs-lookup"><span data-stu-id="b3790-107">Blazor server-side is integrated into [ASP.NET Core Endpoint Routing](xref:fundamentals/routing).</span></span> <span data-ttu-id="b3790-108">Приложение ASP.NET Core настроено для приема входящих подключений для интерактивных компонентов с `MapBlazorHub` в `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="b3790-108">An ASP.NET Core app is configured to accept incoming connections for interactive components with `MapBlazorHub` in `Startup.Configure`:</span></span>

[!code-csharp[](routing/samples_snapshot/3.x/Startup.cs?highlight=5)]

## <a name="route-templates"></a><span data-ttu-id="b3790-109">Шаблоны маршрутов</span><span class="sxs-lookup"><span data-stu-id="b3790-109">Route templates</span></span>

<span data-ttu-id="b3790-110">`Router` Компонент включает маршрутизацию, и для каждого доступного компонента предоставляется шаблон маршрута.</span><span class="sxs-lookup"><span data-stu-id="b3790-110">The `Router` component enables routing, and a route template is provided to each accessible component.</span></span> <span data-ttu-id="b3790-111">Компонент отображается в файле *app. Razor:* `Router`</span><span class="sxs-lookup"><span data-stu-id="b3790-111">The `Router` component appears in the *App.razor* file:</span></span>

<span data-ttu-id="b3790-112">В приложении Блазор на стороне сервера:</span><span class="sxs-lookup"><span data-stu-id="b3790-112">In a Blazor server-side app:</span></span>

```cshtml
<Router AppAssembly="typeof(Startup).Assembly" />
```

<span data-ttu-id="b3790-113">В клиентском приложении Блазор:</span><span class="sxs-lookup"><span data-stu-id="b3790-113">In a Blazor client-side app:</span></span>

```cshtml
<Router AppAssembly="typeof(Program).Assembly" />
```

<span data-ttu-id="b3790-114">При компиляции файла *. Razor* с `@page` директивой создается <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> класс, в котором указан шаблон маршрута.</span><span class="sxs-lookup"><span data-stu-id="b3790-114">When a *.razor* file with an `@page` directive is compiled, the generated class is provided a <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> specifying the route template.</span></span> <span data-ttu-id="b3790-115">Во время выполнения маршрутизатор ищет классы компонентов с `RouteAttribute` помощью и отображает компонент с помощью шаблона маршрута, соответствующего запрошенному URL-адресу.</span><span class="sxs-lookup"><span data-stu-id="b3790-115">At runtime, the router looks for component classes with a `RouteAttribute` and renders the component with a route template that matches the requested URL.</span></span>

<span data-ttu-id="b3790-116">К компоненту можно применить несколько шаблонов маршрутов.</span><span class="sxs-lookup"><span data-stu-id="b3790-116">Multiple route templates can be applied to a component.</span></span> <span data-ttu-id="b3790-117">Следующий компонент отвечает на запросы `/BlazorRoute` и: `/DifferentBlazorRoute`</span><span class="sxs-lookup"><span data-stu-id="b3790-117">The following component responds to requests for `/BlazorRoute` and `/DifferentBlazorRoute`:</span></span>

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/BlazorRoute.razor?name=snippet_BlazorRoute)]

> [!IMPORTANT]
> <span data-ttu-id="b3790-118">Для правильной генерации маршрутов приложение `<base>` должно включать тег в файл *wwwroot/index.HTML* (на стороне клиента блазор) или Pages */_Host. cshtml* (блазор на стороне сервера) с базовым путем к `href` приложению, указанным в атрибуте ( `<base href="/">`).</span><span class="sxs-lookup"><span data-stu-id="b3790-118">To generate routes properly, the app must include a `<base>` tag in its *wwwroot/index.html* file (Blazor client-side) or *Pages/_Host.cshtml* file (Blazor server-side) with the app base path specified in the `href` attribute (`<base href="/">`).</span></span> <span data-ttu-id="b3790-119">Дополнительные сведения см. в разделе <xref:host-and-deploy/blazor/client-side#app-base-path>.</span><span class="sxs-lookup"><span data-stu-id="b3790-119">For more information, see <xref:host-and-deploy/blazor/client-side#app-base-path>.</span></span>

## <a name="provide-custom-content-when-content-isnt-found"></a><span data-ttu-id="b3790-120">Указать пользовательское содержимое, когда содержимое не найдено</span><span class="sxs-lookup"><span data-stu-id="b3790-120">Provide custom content when content isn't found</span></span>

<span data-ttu-id="b3790-121">`Router` Компонент позволяет приложению указать пользовательское содержимое, если содержимое для запрошенного маршрута не найдено.</span><span class="sxs-lookup"><span data-stu-id="b3790-121">The `Router` component allows the app to specify custom content if content isn't found for the requested route.</span></span>

<span data-ttu-id="b3790-122">В файле *app. Razor* задайте пользовательское содержимое в `<NotFoundContent>` элементе `Router` компонента:</span><span class="sxs-lookup"><span data-stu-id="b3790-122">In the *App.razor* file, set custom content in the `<NotFoundContent>` element of the `Router` component:</span></span>

```cshtml
<Router AppAssembly="typeof(Startup).Assembly">
    <NotFoundContent>
        <h1>Sorry</h1>
        <p>Sorry, there's nothing at this address.</p> b
    </NotFoundContent>
</Router>
```

<span data-ttu-id="b3790-123">Содержимое `<NotFoundContent>` может включать произвольные элементы, например другие интерактивные компоненты.</span><span class="sxs-lookup"><span data-stu-id="b3790-123">The content of `<NotFoundContent>` can include arbitrary items, such as other interactive components.</span></span>

## <a name="route-parameters"></a><span data-ttu-id="b3790-124">Параметры маршрута</span><span class="sxs-lookup"><span data-stu-id="b3790-124">Route parameters</span></span>

<span data-ttu-id="b3790-125">Маршрутизатор использует параметры маршрута для заполнения соответствующих параметров компонента с тем же именем (без учета регистра):</span><span class="sxs-lookup"><span data-stu-id="b3790-125">The router uses route parameters to populate the corresponding component parameters with the same name (case insensitive):</span></span>

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/RouteParameter.razor?name=snippet_RouteParameter&highlight=2,7-8)]

<span data-ttu-id="b3790-126">Необязательные параметры не поддерживаются для приложений Блазор в предварительной версии ASP.NET Core 3,0.</span><span class="sxs-lookup"><span data-stu-id="b3790-126">Optional parameters aren't supported for Blazor apps in ASP.NET Core 3.0 Preview.</span></span> <span data-ttu-id="b3790-127">В `@page` предыдущем примере применяются две директивы.</span><span class="sxs-lookup"><span data-stu-id="b3790-127">Two `@page` directives are applied in the previous example.</span></span> <span data-ttu-id="b3790-128">Первый позволяет переходить к компоненту без параметра.</span><span class="sxs-lookup"><span data-stu-id="b3790-128">The first permits navigation to the component without a parameter.</span></span> <span data-ttu-id="b3790-129">Вторая `@page` директива `{text}` принимает параметр Route и присваивает значение `Text` свойству.</span><span class="sxs-lookup"><span data-stu-id="b3790-129">The second `@page` directive takes the `{text}` route parameter and assigns the value to the `Text` property.</span></span>

## <a name="route-constraints"></a><span data-ttu-id="b3790-130">Ограничения маршрута</span><span class="sxs-lookup"><span data-stu-id="b3790-130">Route constraints</span></span>

<span data-ttu-id="b3790-131">Ограничение маршрута применяет сопоставление типов в сегменте маршрута к компоненту.</span><span class="sxs-lookup"><span data-stu-id="b3790-131">A route constraint enforces type matching on a route segment to a component.</span></span>

<span data-ttu-id="b3790-132">В следующем примере маршрут к `Users` компоненту соответствует только в том случае, если:</span><span class="sxs-lookup"><span data-stu-id="b3790-132">In the following example, the route to the `Users` component only matches if:</span></span>

* <span data-ttu-id="b3790-133">В URL-адресе запроса имеется сегмент маршрута.`Id`</span><span class="sxs-lookup"><span data-stu-id="b3790-133">An `Id` route segment is present on the request URL.</span></span>
* <span data-ttu-id="b3790-134">Сегмент является целым числом (`int`). `Id`</span><span class="sxs-lookup"><span data-stu-id="b3790-134">The `Id` segment is an integer (`int`).</span></span>

[!code-cshtml[](routing/samples_snapshot/3.x/Constraint.razor?highlight=1)]

<span data-ttu-id="b3790-135">Ограничения маршрутов, приведенные в следующей таблице, доступны.</span><span class="sxs-lookup"><span data-stu-id="b3790-135">The route constraints shown in the following table are available.</span></span> <span data-ttu-id="b3790-136">Сведения об ограничениях маршрута, соответствующих инвариантному языку и региональным параметрам, см. в предупреждении ниже таблицы для получения дополнительных сведений.</span><span class="sxs-lookup"><span data-stu-id="b3790-136">For the route constraints that match with the invariant culture, see the warning below the table for more information.</span></span>

| <span data-ttu-id="b3790-137">Ограничение</span><span class="sxs-lookup"><span data-stu-id="b3790-137">Constraint</span></span> | <span data-ttu-id="b3790-138">Пример</span><span class="sxs-lookup"><span data-stu-id="b3790-138">Example</span></span>           | <span data-ttu-id="b3790-139">Примеры совпадений</span><span class="sxs-lookup"><span data-stu-id="b3790-139">Example Matches</span></span>                                                                  | <span data-ttu-id="b3790-140">Инвариант</span><span class="sxs-lookup"><span data-stu-id="b3790-140">Invariant</span></span><br><span data-ttu-id="b3790-141">язык и региональные параметры</span><span class="sxs-lookup"><span data-stu-id="b3790-141">culture</span></span><br><span data-ttu-id="b3790-142">соответствие</span><span class="sxs-lookup"><span data-stu-id="b3790-142">matching</span></span> |
| ---------- | ----------------- | -------------------------------------------------------------------------------- | :------------------------------: |
| `bool`     | `{active:bool}`   | <span data-ttu-id="b3790-143">`true`, `FALSE`</span><span class="sxs-lookup"><span data-stu-id="b3790-143">`true`, `FALSE`</span></span>                                                                  | <span data-ttu-id="b3790-144">Нет</span><span class="sxs-lookup"><span data-stu-id="b3790-144">No</span></span>                               |
| `datetime` | `{dob:datetime}`  | <span data-ttu-id="b3790-145">`2016-12-31`, `2016-12-31 7:32pm`</span><span class="sxs-lookup"><span data-stu-id="b3790-145">`2016-12-31`, `2016-12-31 7:32pm`</span></span>                                                | <span data-ttu-id="b3790-146">Да</span><span class="sxs-lookup"><span data-stu-id="b3790-146">Yes</span></span>                              |
| `decimal`  | `{price:decimal}` | <span data-ttu-id="b3790-147">`49.99`, `-1,000.01`</span><span class="sxs-lookup"><span data-stu-id="b3790-147">`49.99`, `-1,000.01`</span></span>                                                             | <span data-ttu-id="b3790-148">Да</span><span class="sxs-lookup"><span data-stu-id="b3790-148">Yes</span></span>                              |
| `double`   | `{weight:double}` | <span data-ttu-id="b3790-149">`1.234`, `-1,001.01e8`</span><span class="sxs-lookup"><span data-stu-id="b3790-149">`1.234`, `-1,001.01e8`</span></span>                                                           | <span data-ttu-id="b3790-150">Да</span><span class="sxs-lookup"><span data-stu-id="b3790-150">Yes</span></span>                              |
| `float`    | `{weight:float}`  | <span data-ttu-id="b3790-151">`1.234`, `-1,001.01e8`</span><span class="sxs-lookup"><span data-stu-id="b3790-151">`1.234`, `-1,001.01e8`</span></span>                                                           | <span data-ttu-id="b3790-152">Да</span><span class="sxs-lookup"><span data-stu-id="b3790-152">Yes</span></span>                              |
| `guid`     | `{id:guid}`       | <span data-ttu-id="b3790-153">`CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}`</span><span class="sxs-lookup"><span data-stu-id="b3790-153">`CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}`</span></span> | <span data-ttu-id="b3790-154">Нет</span><span class="sxs-lookup"><span data-stu-id="b3790-154">No</span></span>                               |
| `int`      | `{id:int}`        | <span data-ttu-id="b3790-155">`123456789`, `-123456789`</span><span class="sxs-lookup"><span data-stu-id="b3790-155">`123456789`, `-123456789`</span></span>                                                        | <span data-ttu-id="b3790-156">Да</span><span class="sxs-lookup"><span data-stu-id="b3790-156">Yes</span></span>                              |
| `long`     | `{ticks:long}`    | <span data-ttu-id="b3790-157">`123456789`, `-123456789`</span><span class="sxs-lookup"><span data-stu-id="b3790-157">`123456789`, `-123456789`</span></span>                                                        | <span data-ttu-id="b3790-158">Да</span><span class="sxs-lookup"><span data-stu-id="b3790-158">Yes</span></span>                              |

> [!WARNING]
> <span data-ttu-id="b3790-159">Ограничения маршрута, которые проверяют URL-адрес и могут быть преобразованы в тип CLR (например, `int` или `DateTime`), всегда используют инвариантные язык и региональные параметры.</span><span class="sxs-lookup"><span data-stu-id="b3790-159">Route constraints that verify the URL and are converted to a CLR type (such as `int` or `DateTime`) always use the invariant culture.</span></span> <span data-ttu-id="b3790-160">Эти ограничения предполагают, что URL-адрес является нелокализуемым.</span><span class="sxs-lookup"><span data-stu-id="b3790-160">These constraints assume that the URL is non-localizable.</span></span>

## <a name="navlink-component"></a><span data-ttu-id="b3790-161">Компонент Навлинк</span><span class="sxs-lookup"><span data-stu-id="b3790-161">NavLink component</span></span>

<span data-ttu-id="b3790-162">Используйте компонент вместо элементов HTML-гиперссылок (`<a>`) при создании навигационных ссылок. `NavLink`</span><span class="sxs-lookup"><span data-stu-id="b3790-162">Use a `NavLink` component in place of HTML hyperlink elements (`<a>`) when creating navigation links.</span></span> <span data-ttu-id="b3790-163">Компонент ведет себя `<a>` как элемент, `active` за исключением того, что он переключает класс CSS на основе того, `href` соответствует ли он текущему URL-адресу. `NavLink`</span><span class="sxs-lookup"><span data-stu-id="b3790-163">A `NavLink` component behaves like an `<a>` element, except it toggles an `active` CSS class based on whether its `href` matches the current URL.</span></span> <span data-ttu-id="b3790-164">`active` Класс помогает пользователю понять, какая страница является активной страницей между отображаемыми ссылками навигации.</span><span class="sxs-lookup"><span data-stu-id="b3790-164">The `active` class helps a user understand which page is the active page among the navigation links displayed.</span></span>

<span data-ttu-id="b3790-165">Следующий `NavMenu` компонент создает панель навигации [начальной загрузки](https://getbootstrap.com/docs/) , которая демонстрирует использование `NavLink` компонентов.</span><span class="sxs-lookup"><span data-stu-id="b3790-165">The following `NavMenu` component creates a [Bootstrap](https://getbootstrap.com/docs/) navigation bar that demonstrates how to use `NavLink` components:</span></span>

[!code-cshtml[](routing/samples_snapshot/3.x/NavMenu.razor?highlight=4,9)]

<span data-ttu-id="b3790-166">`Match` `NavLinkMatch` Атрибут`<NavLink>` элемента можно назначить двумя способами:</span><span class="sxs-lookup"><span data-stu-id="b3790-166">There are two `NavLinkMatch` options that you can assign to the `Match` attribute of the `<NavLink>` element:</span></span>

* <span data-ttu-id="b3790-167">`NavLinkMatch.All`&ndash; Параметр`NavLink` активен, если он соответствует всему текущему URL-адресу.</span><span class="sxs-lookup"><span data-stu-id="b3790-167">`NavLinkMatch.All` &ndash; The `NavLink` is active when it matches the entire current URL.</span></span>
* <span data-ttu-id="b3790-168">`NavLinkMatch.Prefix`(*по умолчанию*) Параметр активен, `NavLink` если он соответствует любому префиксу текущего URL-адреса. &ndash;</span><span class="sxs-lookup"><span data-stu-id="b3790-168">`NavLinkMatch.Prefix` (*default*) &ndash; The `NavLink` is active when it matches any prefix of the current URL.</span></span>

<span data-ttu-id="b3790-169">В предыдущем примере домашняя `NavLink` `href=""` страница совпадает с домашним `active` URL-адресом и получает класс CSS только в URL-адресе базового пути по умолчанию `https://localhost:5001/`для приложения (например,).</span><span class="sxs-lookup"><span data-stu-id="b3790-169">In the preceding example, the Home `NavLink` `href=""` matches the home URL and only receives the `active` CSS class at the app's default base path URL (for example, `https://localhost:5001/`).</span></span> <span data-ttu-id="b3790-170">Второй `NavLink` получает класс, `active` когда пользователь `MyComponent` посещает любой URL-адрес с префиксом (например, `https://localhost:5001/MyComponent` и `https://localhost:5001/MyComponent/AnotherSegment`).</span><span class="sxs-lookup"><span data-stu-id="b3790-170">The second `NavLink` receives the `active` class when the user visits any URL with a `MyComponent` prefix (for example, `https://localhost:5001/MyComponent` and `https://localhost:5001/MyComponent/AnotherSegment`).</span></span>

## <a name="uri-and-navigation-state-helpers"></a><span data-ttu-id="b3790-171">URI и вспомогательные функции состояния навигации</span><span class="sxs-lookup"><span data-stu-id="b3790-171">URI and navigation state helpers</span></span>

<span data-ttu-id="b3790-172">Используется `Microsoft.AspNetCore.Components.IUriHelper` для работы с URI и навигацией C# в коде.</span><span class="sxs-lookup"><span data-stu-id="b3790-172">Use `Microsoft.AspNetCore.Components.IUriHelper` to work with URIs and navigation in C# code.</span></span> <span data-ttu-id="b3790-173">`IUriHelper`предоставляет события и методы, приведенные в следующей таблице.</span><span class="sxs-lookup"><span data-stu-id="b3790-173">`IUriHelper` provides the event and methods shown in the following table.</span></span>

| <span data-ttu-id="b3790-174">Член</span><span class="sxs-lookup"><span data-stu-id="b3790-174">Member</span></span> | <span data-ttu-id="b3790-175">Описание</span><span class="sxs-lookup"><span data-stu-id="b3790-175">Description</span></span> |
| ------ | ----------- |
| `GetAbsoluteUri` | <span data-ttu-id="b3790-176">Возвращает текущий абсолютный URI.</span><span class="sxs-lookup"><span data-stu-id="b3790-176">Gets the current absolute URI.</span></span> |
| `GetBaseUri` | <span data-ttu-id="b3790-177">Получает базовый URI (с завершающей косой чертой), который можно добавить в начало относительных путей URI для получения абсолютного URI.</span><span class="sxs-lookup"><span data-stu-id="b3790-177">Gets the base URI (with a trailing slash) that can be prepended to relative URI paths to produce an absolute URI.</span></span> <span data-ttu-id="b3790-178">Как правило `GetBaseUri` , соответствует `href`атрибуту элемента документа в wwwroot/index.HTML (на стороне клиента блазор) или *pages/_Host. cshtml* (блазор на стороне сервера). `<base>`</span><span class="sxs-lookup"><span data-stu-id="b3790-178">Typically, `GetBaseUri` corresponds to the `href` attribute on the document's `<base>` element in *wwwroot/index.html* (Blazor client-side) or *Pages/_Host.cshtml* (Blazor server-side).</span></span> |
| `NavigateTo` | <span data-ttu-id="b3790-179">Переходит по указанному универсальному коду ресурса (URI).</span><span class="sxs-lookup"><span data-stu-id="b3790-179">Navigates to the specified URI.</span></span> <span data-ttu-id="b3790-180">Если `forceLoad` имеет `true`значение:</span><span class="sxs-lookup"><span data-stu-id="b3790-180">If `forceLoad` is `true`:</span></span><ul><li><span data-ttu-id="b3790-181">Маршрутизация на стороне клиента обходится.</span><span class="sxs-lookup"><span data-stu-id="b3790-181">Client-side routing is bypassed.</span></span></li><li><span data-ttu-id="b3790-182">Браузер вынужден загрузить новую страницу с сервера, независимо от того, обрабатывается ли универсальный код ресурса клиентским маршрутизатором.</span><span class="sxs-lookup"><span data-stu-id="b3790-182">The browser is forced to load the new page from the server, whether or not the URI is normally handled by the client-side router.</span></span></li></ul> |
| `OnLocationChanged` | <span data-ttu-id="b3790-183">Событие, возникающее при изменении расположения навигации.</span><span class="sxs-lookup"><span data-stu-id="b3790-183">An event that fires when the navigation location has changed.</span></span> |
| `ToAbsoluteUri` | <span data-ttu-id="b3790-184">Преобразует относительный URI в абсолютный URI.</span><span class="sxs-lookup"><span data-stu-id="b3790-184">Converts a relative URI into an absolute URI.</span></span> |
| `ToBaseRelativePath` | <span data-ttu-id="b3790-185">Учитывая базовый URI (например, URI, ранее возвращенный `GetBaseUri`), преобразует абсолютный URI в универсальный код ресурса (URI) по отношению к базовому префиксу URI.</span><span class="sxs-lookup"><span data-stu-id="b3790-185">Given a base URI (for example, a URI previously returned by `GetBaseUri`), converts an absolute URI into a URI relative to the base URI prefix.</span></span> |

<span data-ttu-id="b3790-186">Следующий компонент переходит к `Counter` компоненту приложения при выборе этой кнопки:</span><span class="sxs-lookup"><span data-stu-id="b3790-186">The following component navigates to the app's `Counter` component when the button is selected:</span></span>

```cshtml
@page "/navigate"
@using Microsoft.AspNetCore.Components
@inject IUriHelper UriHelper

<h1>Navigate in Code Example</h1>

<button class="btn btn-primary" @onclick="NavigateToCounterComponent">
    Navigate to the Counter component
</button>

@code {
    private void NavigateToCounterComponent()
    {
        UriHelper.NavigateTo("counter");
    }
}
```