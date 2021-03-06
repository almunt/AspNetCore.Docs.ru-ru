---
title: Ограничить время существования защищенных полезных данных в ASP.NET Core
author: rick-anderson
description: Узнайте, как ограничить время существования защищенных полезных данных с помощью ASP.NET Core API-интерфейсов защиты данных.
ms.author: riande
ms.date: 10/14/2016
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
uid: security/data-protection/consumer-apis/limited-lifetime-payloads
ms.openlocfilehash: f76aca460c293b5f814ba10ee6c8ac68b3d147bb
ms.sourcegitcommit: 65add17f74a29a647d812b04517e46cbc78258f9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/19/2020
ms.locfileid: "88634427"
---
# <a name="limit-the-lifetime-of-protected-payloads-in-aspnet-core"></a>Ограничить время существования защищенных полезных данных в ASP.NET Core

Существуют сценарии, в которых разработчик приложения хочет создать защищенные полезные данные, срок действия которых истекает через заданный период времени. Например, защищенные полезные данные могут представлять маркер сброса пароля, который должен быть допустимым только в течение одного часа. Разработчику наверняка можно создать собственный формат полезных данных, который содержит внедренную дату истечения срока действия, и опытные разработчики могут сделать это все в любом случае, но для большинства разработчиков, занимающихся обработкой этих сроков, может расти утомительно.

Чтобы упростить эту задачу для нашей аудитории разработчика, пакет [Microsoft. AspNetCore. данные защиты. Extensions](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Extensions/) содержит служебные API-интерфейсы для создания полезных данных, срок действия которых истекает автоматически по истечении заданного периода времени. Эти API зависает от `ITimeLimitedDataProtector` типа.

## <a name="api-usage"></a>Использование API

`ITimeLimitedDataProtector`Интерфейс — это основной интерфейс для защиты и снятия защиты полезных данных с ограниченным временем или с самостоятельным истечением срока действия. Чтобы создать экземпляр класса `ITimeLimitedDataProtector` , сначала требуется экземпляр регулярного [идатапротектор](xref:security/data-protection/consumer-apis/overview) , созданный с определенной целью. После того как `IDataProtector` экземпляр доступен, вызовите `IDataProtector.ToTimeLimitedDataProtector` метод расширения для возврата предохранителя с помощью встроенных возможностей истечения срока действия.

`ITimeLimitedDataProtector` предоставляет следующие интерфейсы API и интерфейсов расширения:

* Креатепротектор (Назначение строки): Итимелимитеддатапротектор — этот API аналогичен существующему `IDataProtectionProvider.CreateProtector` в том, что его можно использовать для создания [цепочек целей](xref:security/data-protection/consumer-apis/purpose-strings) из корневого средства защиты с ограниченным сроком действия.

* Защита (незашифрованный текст в байтах, срок действия DateTimeOffset): Byte []

* Защитить (байт [] в формате UTC, время существования TimeSpan): Byte []

* Защитить (байт [] в формате UTC): Byte []

* Защита (обычный текст строки, срок действия DateTimeOffset): строка

* Защита (обычный текст в виде строки, время существования TimeSpan): строка

* Защита (текст в виде строки): строка

В дополнение к основным `Protect` методам, которые принимают только открытый текст, существуют новые перегрузки, которые позволяют указать дату истечения срока действия полезных данных. Дата истечения срока действия может быть задана как абсолютная дата (через `DateTimeOffset` ) или как относительное время (от текущего системного времени через `TimeSpan` ). Если вызывается перегрузка, которая не принимает истечение срока действия, то предполагается, что полезная нагрузка никогда не истечет.

* Снять защиту (Byte [] protectedData, исходящий срок действия DateTimeOffset): Byte []

* Снять защиту (байт [] protectedData): Byte []

* Снять защиту (строковый protectedData, исходящий срок действия DateTimeOffset): строка

* Снять защиту (строка protectedData): строка

`Unprotect`Методы возвращают исходные незащищенные данные. Если еще не истек срок действия полезных данных, абсолютный срок действия возвращается как необязательный параметр out вместе с исходными незащищенными данными. Если срок действия полезных данных истек, все перегрузки метода Unprotect будут вызывать CryptographicException.

>[!WARNING]
> Не рекомендуется использовать эти API для защиты полезных данных, требующих долгосрочного или неопределенного сохраняемости. «Могу ли я позволить необратимому восстановлению защищенных полезных данных через месяц?» может служить хорошим правилом для Thumb. Если ответ — нет, разработчики должны рассмотреть альтернативные API.

В приведенном ниже примере используются [пути кода без di](xref:security/data-protection/configuration/non-di-scenarios) для создания экземпляра системы защиты данных. Чтобы запустить этот пример, убедитесь, что предварительно добавлены ссылки на пакет Microsoft. AspNetCore. MDAC. Extensions.

[!code-csharp[](limited-lifetime-payloads/samples/limitedlifetimepayloads.cs)]
