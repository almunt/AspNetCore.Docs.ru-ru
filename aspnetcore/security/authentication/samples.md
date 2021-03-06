---
title: Примеры проверки подлинности для ASP.NET Core
author: rick-anderson
description: Содержит ссылки на примеры проверки подлинности в репозитории ASP.NET Core.
ms.author: riande
ms.date: 01/31/2019
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
uid: security/authentication/samples
ms.openlocfilehash: 290c956b2035e47e5b34dba15fbec665461dd94a
ms.sourcegitcommit: 65add17f74a29a647d812b04517e46cbc78258f9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/19/2020
ms.locfileid: "88630748"
---
# <a name="authentication-samples-for-aspnet-core"></a>Примеры проверки подлинности для ASP.NET Core

Автор: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)

::: moniker range=">= aspnetcore-3.0"

[Репозиторий ASP.NET Core](https://github.com/dotnet/AspNetCore) содержит следующие примеры проверки подлинности в папке *AspNetCore/src/Security/Samples* :

* [Преобразование утверждений](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/ClaimsTransformation)
* [Cookie идентификаци](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/Cookies)
* [Поставщик настраиваемой политики — Иаусоризатионполиципровидер](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/CustomPolicyProvider)
* [Схемы и параметры динамической проверки подлинности](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/DynamicSchemes)
* [Внешние утверждения](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/Identity.ExternalClaims)
* [Выбор между cookie и другой схемой проверки подлинности на основе запроса](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/PathSchemeSelection)
* [Разрешает доступ к статическим файлам](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/StaticFilesAuth)

## <a name="run-the-samples"></a>Запуск шаблонов

* Выберите [ветвь](https://github.com/dotnet/AspNetCore). Например `release/3.1`.
* Клонировать или скачать [репозиторий ASP.NET Core](https://github.com/dotnet/AspNetCore).
* Убедитесь, что установлена версия [пакет SDK для .NET Core](https://dotnet.microsoft.com/download/dotnet-core) , соответствующая клону репозитория ASP.NET Core.
* Перейдите к примеру в *AspNetCore/src/Security/Samples* и запустите пример с помощью `dotnet run` .

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[Репозиторий ASP.NET Core](https://github.com/dotnet/AspNetCore) содержит следующие примеры проверки подлинности в папке *AspNetCore/src/Security/Samples* :

* [Преобразование утверждений](https://github.com/dotnet/AspNetCore/tree/release/2.1/src/Security/samples/ClaimsTransformation)
* [Cookie идентификаци](https://github.com/dotnet/AspNetCore/tree/release/2.1/src/Security/samples/Cookies)
* [Поставщик настраиваемой политики — Иаусоризатионполиципровидер](https://github.com/dotnet/AspNetCore/tree/2.1.3/src/Security/samples/CustomPolicyProvider)
* [Схемы и параметры динамической проверки подлинности](https://github.com/dotnet/AspNetCore/tree/release/2.1/src/Security/samples/DynamicSchemes)
* [Внешние утверждения](https://github.com/dotnet/AspNetCore/tree/release/2.1/src/Security/samples/Identity.ExternalClaims)
* [Выбор между cookie и другой схемой проверки подлинности на основе запроса](https://github.com/dotnet/AspNetCore/tree/release/2.1/src/Security/samples/PathSchemeSelection)
* [Разрешает доступ к статическим файлам](https://github.com/dotnet/AspNetCore/tree/2.1.3/src/Security/samples/StaticFilesAuth)

## <a name="run-the-samples"></a>Запуск шаблонов

* Выберите [ветвь](https://github.com/dotnet/AspNetCore). Например `release/2.1`.
* Клонировать или скачать [репозиторий ASP.NET Core](https://github.com/dotnet/AspNetCore).
* Убедитесь, что установлена версия [пакет SDK для .NET Core](https://dotnet.microsoft.com/download/dotnet-core) , соответствующая клону репозитория ASP.NET Core.
* Перейдите к примеру в *AspNetCore/src/Security/Samples* и запустите пример с помощью `dotnet run` .

::: moniker-end
