---
title: Хэширование паролей в ASP.NET Core
author: rick-anderson
description: Узнайте, как хэшировать пароли с помощью ASP.NET Core интерфейсов API защиты данных.
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
uid: security/data-protection/consumer-apis/password-hashing
ms.openlocfilehash: 19263400397a9dfe2d9e6044109d6d063023f6f4
ms.sourcegitcommit: 65add17f74a29a647d812b04517e46cbc78258f9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/19/2020
ms.locfileid: "88629864"
---
# <a name="hash-passwords-in-aspnet-core"></a>Хэширование паролей в ASP.NET Core

База кода защиты данных включает пакет *Microsoft. AspNetCore. Cryptography. KeyDerivation* , который содержит функции формирования ключа шифрования. Этот пакет является автономным компонентом и не зависит от остальной части системы защиты данных. Его можно использовать полностью независимо друг от друга. Для удобства источник существует рядом с базой кода защиты данных.

В настоящее время пакет содержит метод `KeyDerivation.Pbkdf2` , позволяющий хэшировать пароль с помощью [алгоритма PBKDF2](https://tools.ietf.org/html/rfc2898#section-5.2). Этот API очень похож на существующий [тип Rfc2898DeriveBytes](/dotnet/api/system.security.cryptography.rfc2898derivebytes).NET Framework, но есть три важных различия:

1. `KeyDerivation.Pbkdf2`Метод поддерживает использование нескольких прфс (в настоящее время `HMACSHA1` , `HMACSHA256` и `HMACSHA512` ), в то время как `Rfc2898DeriveBytes` тип поддерживает только `HMACSHA1` .

2. `KeyDerivation.Pbkdf2`Метод обнаруживает текущую операционную систему и пытается выбрать наиболее оптимизированную реализацию подпрограммы, предоставляя гораздо более высокую производительность в некоторых случаях. (В Windows 8 она предлагает примерно 10 раз пропускную способность `Rfc2898DeriveBytes` .)

3. `KeyDerivation.Pbkdf2`Метод требует, чтобы вызывающий объект указал все параметры (соль, PRF и число итераций). `Rfc2898DeriveBytes`Тип предоставляет значения по умолчанию для этих параметров.

[!code-csharp[](password-hashing/samples/passwordhasher.cs)]

Ознакомьтесь с типом [исходного кода](https://github.com/dotnet/AspNetCore/blob/master/src/Identity/Extensions.Core/src/PasswordHasher.cs) для ASP.NET Core Identity `PasswordHasher` реального варианта использования.
