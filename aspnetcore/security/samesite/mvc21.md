---
title: Пример SameSite ASP.NET Core 2,1 MVC cookie
author: rick-anderson
description: Пример SameSite ASP.NET Core 2,1 MVC cookie
monikerRange: '>= aspnetcore-2.1 < aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 12/03/2019
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
uid: security/samesite/mvc21
ms.openlocfilehash: 0a719ae48199f7854ded534446045eb304d4d9f0
ms.sourcegitcommit: 65add17f74a29a647d812b04517e46cbc78258f9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/19/2020
ms.locfileid: "88632360"
---
# <a name="aspnet-core-21-mvc-samesite-no-loccookie-sample"></a>Пример SameSite ASP.NET Core 2,1 MVC cookie

ASP.NET Core 2,1 имеет встроенную поддержку атрибута [SameSite](https://www.owasp.org/index.php/SameSite) , но она была записана в исходный стандарт. [Исправленное поведение](https://github.com/dotnet/aspnetcore/issues/8212) изменило значение параметра `SameSite.None` , чтобы выдать атрибут sameSite со значением `None` , а не выдавать значение вообще. Если вы не хотите выпустить значение, можно задать `SameSite` для свойства значение cookie -1.

[!INCLUDE[](~/includes/SameSiteIdentity.md)]

## <a name="writing-the-samesite-attribute"></a><a name="sampleCode"></a>Написание атрибута SameSite

Ниже приведен пример того, как записать атрибут SameSite в cookie :

```c#
var cookieOptions = new CookieOptions
{
    // Set the secure flag, which Chrome's changes will require for SameSite none.
    // Note this will also require you to be running on HTTPS
    Secure = true,

    // Set the cookie to HTTP only which is good practice unless you really do need
    // to access it client side in scripts.
    HttpOnly = true,

    // Add the SameSite attribute, this will emit the attribute with a value of none.
    // To not emit the attribute at all set the SameSite property to (SameSiteMode)(-1).
    SameSite = SameSiteMode.None
};

// Add the cookie to the response cookie collection
Response.Cookies.Append(CookieName, "cookieValue", cookieOptions);
```

## <a name="setting-no-loccookie-authentication-and-session-state-no-loccookies"></a>Настройка Cookie проверки подлинности и состояния сеанса cookie

Cookie Проверка подлинности, состояние сеанса и [различные другие компоненты](https://docs.microsoft.com/aspnet/core/security/samesite?view=aspnetcore-2.1) задают свои параметры sameSite Cookie с помощью параметров, например

```c#
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.Cookie.SameSite = SameSiteMode.None;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.IsEssential = true;
    });

services.AddSession(options =>
{
    options.Cookie.SameSite = SameSiteMode.None;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.IsEssential = true;
});
```

В приведенном выше коде как cookie Проверка подлинности, так и состояние сеанса устанавливают атрибут sameSite в, выдает `None` атрибут со `None` значением, а также присвойте атрибуту Secure значение true.

### <a name="run-the-sample"></a>Запуск примера

При запуске [примера проекта](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC)Загрузите отладчик браузера на начальной странице и используйте его для просмотра cookie коллекции для сайта. Чтобы сделать это в пограничных и Chrome, нажмите на `F12` `Application` вкладку и щелкните URL-адрес сайта в `Cookies` параметре в `Storage` разделе.

![Отладчик браузера::: No-Loc (cookie)::: List](BrowserDebugger.png)

На приведенном выше изображении можно увидеть, что cookie созданный примером при нажатии кнопки "создать SameSite Cookie " имеет значение атрибута SameSite `Lax` , совпадающее со значением, заданным в [образце кода](#sampleCode).

## <a name="intercepting-no-loccookies"></a><a name="interception"></a>Перехват cookie s

Чтобы перехватить cookie , чтобы изменить значение None в соответствии с его поддержкой в агенте браузера пользователя, необходимо использовать по `CookiePolicy` промежуточного слоя. Он должен быть помещен в конвейер HTTP-запросов **перед** всеми компонентами, которые записывают cookie и настраиваются в `ConfigureServices()` .

Чтобы вставить его в конвейер, используйте `app.UseCookiePolicy()` `Configure(IApplicationBuilder, IHostingEnvironment)` метод в [Startup.CS](https://github.com/blowdart/AspNetSameSiteSamples/blob/master/AspNetCore21MVC/Startup.cs). Пример:

```c#
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
       app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseAuthentication();
    app.UseSession();

    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });
}
```

Затем в `ConfigureServices(IServiceCollection services)` cookie политике настройте политику для вызова вспомогательного класса, когда cookie добавляются или удаляются. Пример:

```c#
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<CookiePolicyOptions>(options =>
    {
        options.CheckConsentNeeded = context => true;
        options.MinimumSameSitePolicy = SameSiteMode.None;
        options.OnAppendCookie = cookieContext =>
            CheckSameSite(cookieContext.Context, cookieContext.CookieOptions);
        options.OnDeleteCookie = cookieContext =>
            CheckSameSite(cookieContext.Context, cookieContext.CookieOptions);
    });
}

private void CheckSameSite(HttpContext httpContext, CookieOptions options)
{
    if (options.SameSite == SameSiteMode.None)
    {
        var userAgent = httpContext.Request.Headers["User-Agent"].ToString();
        if (SameSite.BrowserDetection.DisallowsSameSiteNone(userAgent))
        {
            options.SameSite = (SameSiteMode)(-1);
        }
    }
}
```

Вспомогательная функция `CheckSameSite(HttpContext, CookieOptions)` :

* Вызывается, когда cookie s добавляется к запросу или удаляется из запроса.
* Проверяет, `SameSite` имеет ли свойство значение `None` .
* Если параметр `SameSite` имеет значение `None` , а для текущего агента пользователя известно, что он не поддерживает значение атрибута None. Проверка выполняется с помощью класса [самеситесуппорт](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/samesite/sample/snippets/SameSiteSupport.cs) :
  * Задает, чтобы не выдавало `SameSite` значение, задав для свойства `(SameSiteMode)(-1)`

## <a name="targeting-net-framework"></a>Нацеливание на .NET Framework

ASP.NET Core и System. Web (классическая модель ASP.NET) имеют независимые реализации SameSite. Исправления SameSite KB для .NET Framework не требуются, если используется ASP.NET Core и не является требованием к версии System. Web SameSite минимальной платформы (.NET 4.7.2), применяемым к ASP.NET Core.

ASP.NET Core в .NET требует обновления зависимостей пакетов NuGet для получения соответствующих исправлений.

Чтобы получить ASP.NET Core изменения для .NET Framework убедитесь в наличии прямой ссылки на исправленные пакеты и версии (2.1.14 или более поздней версии 2,1).

```xml
<PackageReference Include="Microsoft.Net.Http.Headers" Version="2.1.14" />
<PackageReference Include="Microsoft.AspNetCore.CookiePolicy" Version="2.1.14" />
```

### <a name="more-information"></a>Дополнительные сведения
 
[Обновления Chrome](https://www.chromium.org/updates/same-site) 
 [Документация по](https://docs.microsoft.com/aspnet/core/security/samesite?view=aspnetcore-2.1) 
 ASP.NET Core SameSite [ASP.NET Core 2,1 SameSite объявление об изменениях](https://github.com/dotnet/aspnetcore/issues/8212)