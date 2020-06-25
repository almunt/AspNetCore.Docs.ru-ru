---
title: Защита ASP.NET Core Blazor WebAssembly
author: guardrex
description: Узнайте о защите приложений Blazor WebAssemlby как одностраничных приложений (SPA).
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 06/01/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/security/webassembly/index
ms.openlocfilehash: 877b2bb4b055cca25d64258383cdb39d812e2d6a
ms.sourcegitcommit: 066d66ea150f8aab63f9e0e0668b06c9426296fd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/23/2020
ms.locfileid: "85243243"
---
# <a name="secure-aspnet-core-blazor-webassembly"></a>Защита ASP.NET Core Blazor WebAssembly

Автор: [Javier Calvarro Nelson](https://github.com/javiercn) (Хавьер Кальварро Нельсон)

Защита приложений Blazor WebAssembly обеспечивается аналогично защите одностраничных приложений (SPA). Существует несколько подходов к проверке подлинности пользователей в одностраничных приложениях, но наиболее распространенным и комплексным является использование реализации на основе [протокола OAuth 2.0](https://oauth.net/), например [Open ID Connect (OIDC)](https://openid.net/connect/).

## <a name="authentication-library"></a>Библиотека проверки подлинности

Blazor WebAssembly поддерживает проверку подлинности и авторизацию приложений с использованием OIDC и библиотеки [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication/). Библиотека предоставляет набор примитивов для простой проверки подлинности на серверных компонентах ASP.NET Core. Библиотека включает в себя ASP.NET Core Identity с поддержкой авторизации API на основе [сервера Identity](https://identityserver.io/). Библиотеку можно использовать для проверки подлинности любого стороннего поставщика Identity (IP), который поддерживает OIDC (их называют поставщиками OpenID (OP)).

Проверка подлинности в Blazor WebAssembly реализуется с помощью библиотеки `oidc-client.js`, которая используется для управления сведениями о базовом протоколе проверки подлинности.

Кроме того, доступны другие варианты проверки подлинности одностраничных приложений, например использование файлов cookie SameSite. Однако технический проект BlazorWebAssembly реализован на основе OAuth и OIDC и представляет собой наилучший вариант для проверки подлинности в приложениях Blazor WebAssembly. По соображениям безопасности и в силу функциональных причин вместо [проверки подлинности на основе файлов cookie](xref:security/anti-request-forgery#cookie-based-authentication) была выбрана [проверка подлинности на основе токенов](xref:security/anti-request-forgery#token-based-authentication) на базе [JSON Web Tokens (JWT)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html).

* Использование протокола на основе токенов позволяет уменьшить контактную зону атак, так как токены отправляются не во всех запросах.
* Конечные точки сервера не нуждаются в защите от [подделки межсайтовых запросов (CSRF)](xref:security/anti-request-forgery), так как токены отправляются явным образом. Это позволяет размещать приложения Blazor WebAssembly параллельно с приложениями MVC или Razor Pages.
* Разрешения токенов более узкие, чем разрешения файлов cookie. Например, токены нельзя использовать для управления учетной записью пользователя или изменения пароля пользователя, если такие функции не реализованы явным образом.
* У токенов короткий срок жизни (по умолчанию — один час), что ограничивает распространение атаки. Токены можно отозвать в любое время.
* Автономные токены JWT гарантируют выполнение надлежащего процесса проверки подлинности на клиенте и сервере. Например, на клиенте есть средства для обнаружения и проверки допустимости получаемых токенов, а также подтверждения их выпуска в рамках данного процесса проверки подлинности. Если третья сторона пытается изменить токен в ходе процесса проверки подлинности, клиент может обнаружить измененный токен и не использовать его.
* Токены с OAuth и OIDC обеспечивают безопасность приложения вне зависимости от поведения агента пользователя.
* Протоколы на основе токенов, такие как OAuth и OIDC, позволяют выполнять проверку подлинности и авторизацию размещенных и автономных приложений с одинаковым набором характеристик безопасности.

## <a name="authentication-process-with-oidc"></a>Процесс проверки подлинности с использованием OIDC

Библиотека [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication/) предлагает несколько примитивов для реализации проверки подлинности и авторизации с помощью OIDC. В общих чертах проверка подлинности работает следующим образом.

* Когда анонимный пользователь нажимает кнопку входа или запрашивает страницу с примененным атрибутом [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute), он перенаправляется на страницу входа приложения (`/authentication/login`).
* На странице входа библиотека проверки подлинности готовится к перенаправлению на конечную точку авторизации. Конечная точка авторизации находится вне приложения Blazor WebAssembly и может размещаться в отдельном источнике. Конечная точка определяет, прошел ли пользователь проверку подлинности, и выдает в ответ один или несколько токенов. Библиотека проверки подлинности предоставляет обратный вызов входа для получения ответа проверки подлинности.
  * Если пользователь не прошел проверку подлинности, он перенаправляется в базовую систему проверки подлинности. Обычно это ASP.NET Core Identity.
  * Если пользователь уже прошел проверку подлинности, конечная точка авторизации создает соответствующие токены и перенаправляет браузер назад в конечную точку обратного вызова входа (`/authentication/login-callback`).
* Когда приложение Blazor WebAssembly загружает конечную точку обратного вызова входа (`/authentication/login-callback`), происходит обработка ответа проверки подлинности.
  * Если процесс проверки подлинности завершается успешно, пользователь проходит проверку подлинности и при необходимости перенаправляется по запрошенному исходному защищенному URL-адресу.
  * Если по какой-либо причине проверка подлинности завершается сбоем, пользователь направляется на страницу ошибки входа (`/authentication/login-failed`), и выводится сообщение об ошибке.

## <a name="authorization"></a>Авторизация

В приложениях Blazor WebAssembly авторизацию можно обойти, так как пользователь может изменять весь код на стороне клиента. Это же справедливо для всех технологий на стороне клиента, включая платформы одностраничного приложения JavaScript или собственных приложений для любой операционной системы.

**Всегда выполняйте проверки авторизации на стороне сервера в конечных точках API, к которым обращается клиентское приложение.**

## <a name="refresh-tokens"></a>Маркеры обновления

Маркеры обновления не могут быть защищены на стороне клиента в приложениях Blazor WebAssembly. Поэтому маркеры обновления не должны отправляться в приложение для непосредственного использования.

Маркеры обновления могут поддерживаться и использоваться серверным приложением в размещенном решении Blazor WebAssembly для доступа к интерфейсам API сторонних производителей. Для получения дополнительной информации см. <xref:blazor/security/webassembly/additional-scenarios#authenticate-users-with-a-third-party-provider-and-call-protected-apis-on-the-host-server-and-the-third-party>.

## <a name="implementation-guidance"></a>Рекомендации по реализации

Статьи в этом *обзоре* содержат сведения об аутентификации пользователей в приложениях WebAssembly Blazor для конкретных поставщиков.

Автономные приложения Blazor WebAssembly:

* [Общие рекомендации по использованию поставщиков OIDC и библиотеки проверки подлинности WebAssembly](xref:blazor/security/webassembly/standalone-with-authentication-library)
* [Учетные записи Майкрософт](xref:blazor/security/webassembly/standalone-with-microsoft-accounts)
* [Azure Active Directory (AAD)](xref:blazor/security/webassembly/standalone-with-azure-active-directory)
* [Azure Active Directory (AAD) B2C](xref:blazor/security/webassembly/standalone-with-azure-active-directory-b2c)

Размещенные приложения Blazor WebAssembly:

* [Azure Active Directory (AAD)](xref:blazor/security/webassembly/hosted-with-azure-active-directory)
* [Azure Active Directory (AAD) B2C](xref:blazor/security/webassembly/hosted-with-azure-active-directory-b2c)
* [Сервер Identity](xref:blazor/security/webassembly/hosted-with-identity-server)

Дополнительные рекомендации по настройке см. по этой ссылке: <xref:blazor/security/webassembly/additional-scenarios>.
