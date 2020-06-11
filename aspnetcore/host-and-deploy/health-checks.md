---
title: Проверки работоспособности в ASP.NET Core
author: rick-anderson
description: Узнайте, как настроить проверки работоспособности для инфраструктуры ASP.NET Core, например приложений и баз данных.
monikerRange: '>= aspnetcore-2.2'
ms.author: riande
ms.custom: mvc
ms.date: 12/15/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: host-and-deploy/health-checks
ms.openlocfilehash: cb3ee4f3bf9061d212c1fee85f3f4a22946be097
ms.sourcegitcommit: 6a71b560d897e13ad5b61d07afe4fcb57f8ef6dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/09/2020
ms.locfileid: "84105783"
---
# <a name="health-checks-in-aspnet-core"></a><span data-ttu-id="32395-103">Проверки работоспособности в ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="32395-103">Health checks in ASP.NET Core</span></span>

<span data-ttu-id="32395-104">Автор: [Гленн Кондрон](https://github.com/glennc) (Glenn Condron)</span><span class="sxs-lookup"><span data-stu-id="32395-104">By [Glenn Condron](https://github.com/glennc)</span></span>

::: moniker range=">= aspnetcore-3.0"

<span data-ttu-id="32395-105">ASP.NET Core предоставляет ПО промежуточного слоя и библиотеки для создания отчетов о работоспособности компонентов инфраструктуры приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-105">ASP.NET Core offers Health Checks Middleware and libraries for reporting the health of app infrastructure components.</span></span>

<span data-ttu-id="32395-106">Проверки работоспособности предоставляются приложением как конечные точки HTTP.</span><span class="sxs-lookup"><span data-stu-id="32395-106">Health checks are exposed by an app as HTTP endpoints.</span></span> <span data-ttu-id="32395-107">Конечные точки проверки работоспособности можно настроить для различных сценариев в реальном времени мониторинга:</span><span class="sxs-lookup"><span data-stu-id="32395-107">Health check endpoints can be configured for a variety of real-time monitoring scenarios:</span></span>

* <span data-ttu-id="32395-108">Проверки работоспособности можно использовать с оркестраторами контейнеров и подсистемами балансировки нагрузки, чтобы проверять состояние приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-108">Health probes can be used by container orchestrators and load balancers to check an app's status.</span></span> <span data-ttu-id="32395-109">Например, оркестратор контейнеров может реагировать на неуспешную проверку работоспособности, остановив последовательное развертывание или перезапустив контейнер.</span><span class="sxs-lookup"><span data-stu-id="32395-109">For example, a container orchestrator may respond to a failing health check by halting a rolling deployment or restarting a container.</span></span> <span data-ttu-id="32395-110">Балансировщик нагрузки может реагировать на неработоспособное приложение путем перенаправления трафика от неисправного экземпляра к работающему экземпляру.</span><span class="sxs-lookup"><span data-stu-id="32395-110">A load balancer might react to an unhealthy app by routing traffic away from the failing instance to a healthy instance.</span></span>
* <span data-ttu-id="32395-111">Использование памяти, диска и других ресурсов физического сервера можно отслеживать с точки зрения работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-111">Use of memory, disk, and other physical server resources can be monitored for healthy status.</span></span>
* <span data-ttu-id="32395-112">Проверки работоспособности позволяют проверять зависимости приложения, такие как базы данных и конечные точки внешних служб, чтобы убедиться в доступности и нормальной работе.</span><span class="sxs-lookup"><span data-stu-id="32395-112">Health checks can test an app's dependencies, such as databases and external service endpoints, to confirm availability and normal functioning.</span></span>

<span data-ttu-id="32395-113">[Просмотреть или скачать образец кода](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/health-checks/samples) ([как скачивать](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="32395-113">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/health-checks/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="32395-114">Пример приложения включает примеры сценариев, описанных в этом разделе.</span><span class="sxs-lookup"><span data-stu-id="32395-114">The sample app includes examples of the scenarios described in this topic.</span></span> <span data-ttu-id="32395-115">Чтобы запустить пример приложения для заданного сценария, используйте команду [dotnet run](/dotnet/core/tools/dotnet-run) из папки проекта в командной строке.</span><span class="sxs-lookup"><span data-stu-id="32395-115">To run the sample app for a given scenario, use the [dotnet run](/dotnet/core/tools/dotnet-run) command from the project's folder in a command shell.</span></span> <span data-ttu-id="32395-116">См. файл *README.md* приложения и описания сценариев в этом разделе, чтобы получить сведения о том, как использовать пример приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-116">See the sample app's *README.md* file and the scenario descriptions in this topic for details on how to use the sample app.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="32395-117">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="32395-117">Prerequisites</span></span>

<span data-ttu-id="32395-118">Проверки работоспособности обычно используются в оркестраторе контейнеров или внешней службе мониторинга для проверки состояния приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-118">Health checks are usually used with an external monitoring service or container orchestrator to check the status of an app.</span></span> <span data-ttu-id="32395-119">Перед добавлением проверки работоспособности приложения выберите используемую систему мониторинга.</span><span class="sxs-lookup"><span data-stu-id="32395-119">Before adding health checks to an app, decide on which monitoring system to use.</span></span> <span data-ttu-id="32395-120">Система мониторинга определяет, какие виды проверки работоспособности создавать и как настроить их конечные точки.</span><span class="sxs-lookup"><span data-stu-id="32395-120">The monitoring system dictates what types of health checks to create and how to configure their endpoints.</span></span>

<span data-ttu-id="32395-121">Для приложений ASP.NET Core ссылка на пакет [Microsoft.AspNetCore.Diagnostics.HealthChecks](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.HealthChecks) указывается неявно.</span><span class="sxs-lookup"><span data-stu-id="32395-121">The [Microsoft.AspNetCore.Diagnostics.HealthChecks](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.HealthChecks) package is referenced implicitly for ASP.NET Core apps.</span></span> <span data-ttu-id="32395-122">Чтобы выполнить проверку работоспособности с помощью Entity Framework Core, добавьте ссылку на пакет [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore).</span><span class="sxs-lookup"><span data-stu-id="32395-122">To perform health checks using Entity Framework Core, add a package reference to the [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore) package.</span></span>

<span data-ttu-id="32395-123">В примере приложения приведен код запуска для демонстрации проверки работоспособности для нескольких сценариев.</span><span class="sxs-lookup"><span data-stu-id="32395-123">The sample app provides startup code to demonstrate health checks for several scenarios.</span></span> <span data-ttu-id="32395-124">Сценарий [проверки базы данных](#database-probe) проверяет работоспособность подключения базы данных с помощью [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks).</span><span class="sxs-lookup"><span data-stu-id="32395-124">The [database probe](#database-probe) scenario checks the health of a database connection using [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks).</span></span> <span data-ttu-id="32395-125">Сценарий [проверки DbContext](#entity-framework-core-dbcontext-probe) проверяет базу данных с помощью EF Core `DbContext`.</span><span class="sxs-lookup"><span data-stu-id="32395-125">The [DbContext probe](#entity-framework-core-dbcontext-probe) scenario checks a database using an EF Core `DbContext`.</span></span> <span data-ttu-id="32395-126">Чтобы изучить сценарии для баз данных, пример приложения выполняет следующие действия.</span><span class="sxs-lookup"><span data-stu-id="32395-126">To explore the database scenarios, the sample app:</span></span>

* <span data-ttu-id="32395-127">Создает базу данных и указывает ее строку подключения в файле приложения *appsettings.json*.</span><span class="sxs-lookup"><span data-stu-id="32395-127">Creates a database and provides its connection string in the *appsettings.json* file.</span></span>
* <span data-ttu-id="32395-128">Содержит следующие ссылки на пакеты в своем файле проекта.</span><span class="sxs-lookup"><span data-stu-id="32395-128">Has the following package references in its project file:</span></span>
  * [<span data-ttu-id="32395-129">AspNetCore.HealthChecks.SqlServer</span><span class="sxs-lookup"><span data-stu-id="32395-129">AspNetCore.HealthChecks.SqlServer</span></span>](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/)
  * [<span data-ttu-id="32395-130">Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore</span><span class="sxs-lookup"><span data-stu-id="32395-130">Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore</span></span>](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/)

> [!NOTE]
> <span data-ttu-id="32395-131">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) не поддерживается корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="32395-131">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) isn't maintained or supported by Microsoft.</span></span>

<span data-ttu-id="32395-132">В другом сценарии проверки работоспособности демонстрируется способ фильтрации проверок работоспособности по порту управления.</span><span class="sxs-lookup"><span data-stu-id="32395-132">Another health check scenario demonstrates how to filter health checks to a management port.</span></span> <span data-ttu-id="32395-133">Пример приложения требует создать файл *Properties/launchSettings.json*, который включает в себя URL-адрес управления и порт управления.</span><span class="sxs-lookup"><span data-stu-id="32395-133">The sample app requires you to create a *Properties/launchSettings.json* file that includes the management URL and management port.</span></span> <span data-ttu-id="32395-134">Дополнительные сведения см. в разделе [Фильтр по портам](#filter-by-port).</span><span class="sxs-lookup"><span data-stu-id="32395-134">For more information, see the [Filter by port](#filter-by-port) section.</span></span>

## <a name="basic-health-probe"></a><span data-ttu-id="32395-135">Базовая проверка работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-135">Basic health probe</span></span>

<span data-ttu-id="32395-136">Для многих приложений достаточно конфигурации базовой проверки работоспособности, которая сообщает о доступности приложения для обработки запросов (*жизнеспособности*).</span><span class="sxs-lookup"><span data-stu-id="32395-136">For many apps, a basic health probe configuration that reports the app's availability to process requests (*liveness*) is sufficient to discover the status of the app.</span></span>

<span data-ttu-id="32395-137">Базовая конфигурация регистрирует службы проверки работоспособности и вызывает ПО промежуточного слоя для получения ответа о работоспособности в конечной точке по URL-адресу.</span><span class="sxs-lookup"><span data-stu-id="32395-137">The basic configuration registers health check services and calls the Health Checks Middleware to respond at a URL endpoint with a health response.</span></span> <span data-ttu-id="32395-138">По умолчанию отдельные проверки работоспособности не регистрируются для тестирования какой-либо конкретной зависимости или подсистемы.</span><span class="sxs-lookup"><span data-stu-id="32395-138">By default, no specific health checks are registered to test any particular dependency or subsystem.</span></span> <span data-ttu-id="32395-139">Приложение считается исправным, если оно способно отвечать на запросы по URL-адресу конечной точки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-139">The app is considered healthy if it's capable of responding at the health endpoint URL.</span></span> <span data-ttu-id="32395-140">Модуль записи ответа по умолчанию передает состояние (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) в виде обычного текста в ответе клиенту. Это может быть состояние [HealthStatus.Healthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus), [HealthStatus.Degraded](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) или [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus).</span><span class="sxs-lookup"><span data-stu-id="32395-140">The default response writer writes the status (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) as a plaintext response back to the client, indicating either a [HealthStatus.Healthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus), [HealthStatus.Degraded](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) or [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) status.</span></span>

<span data-ttu-id="32395-141">Зарегистрируйте службы проверки работоспособности при помощи <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> в `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="32395-141">Register health check services with <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="32395-142">Создайте конечную точку проверки работоспособности, вызвав `MapHealthChecks` в `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-142">Create a health check endpoint by calling `MapHealthChecks` in `Startup.Configure`.</span></span>

<span data-ttu-id="32395-143">В примере приложения конечная точка проверки работоспособности создается в `/health` (*BasicStartup.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-143">In the sample app, the health check endpoint is created at `/health` (*BasicStartup.cs*):</span></span>

```csharp
public class BasicStartup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddHealthChecks();
    }

    public void Configure(IApplicationBuilder app)
    {
        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapHealthChecks("/health");
        });
    }
}
```

<span data-ttu-id="32395-144">Чтобы запустить сценарий базовой конфигурации с помощью примера приложения, выполните следующую команду из папки проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-144">To run the basic configuration scenario using the sample app, execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario basic
```

### <a name="docker-example"></a><span data-ttu-id="32395-145">Пример для Docker</span><span class="sxs-lookup"><span data-stu-id="32395-145">Docker example</span></span>

<span data-ttu-id="32395-146">[Docker](xref:host-and-deploy/docker/index) предлагает встроенную директиву `HEALTHCHECK`, которую можно вызвать для проверки состояния приложения, использующего базовую конфигурацию проверки работоспособности:</span><span class="sxs-lookup"><span data-stu-id="32395-146">[Docker](xref:host-and-deploy/docker/index) offers a built-in `HEALTHCHECK` directive that can be used to check the status of an app that uses the basic health check configuration:</span></span>

```
HEALTHCHECK CMD curl --fail http://localhost:5000/health || exit
```

## <a name="create-health-checks"></a><span data-ttu-id="32395-147">Создание проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-147">Create health checks</span></span>

<span data-ttu-id="32395-148">Проверки работоспособности создаются путем реализации интерфейса <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck>.</span><span class="sxs-lookup"><span data-stu-id="32395-148">Health checks are created by implementing the <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> interface.</span></span> <span data-ttu-id="32395-149">Метод <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck.CheckHealthAsync*> возвращает <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult>, указывая состояние работоспособности как `Healthy`, `Degraded` или `Unhealthy`.</span><span class="sxs-lookup"><span data-stu-id="32395-149">The <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck.CheckHealthAsync*> method returns a <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> that indicates the health as `Healthy`, `Degraded`, or `Unhealthy`.</span></span> <span data-ttu-id="32395-150">Результат записывается в ответ в виде обычного текста с кодом состояния, который можно настроить (конфигурация описана в разделе [Варианты проверки работоспособности](#health-check-options)).</span><span class="sxs-lookup"><span data-stu-id="32395-150">The result is written as a plaintext response with a configurable status code (configuration is described in the [Health check options](#health-check-options) section).</span></span> <span data-ttu-id="32395-151"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> также может возвращать необязательные пары "ключ-значение".</span><span class="sxs-lookup"><span data-stu-id="32395-151"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> can also return optional key-value pairs.</span></span>

<span data-ttu-id="32395-152">Следующий класс `ExampleHealthCheck` демонстрирует структуру процесса проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-152">The following `ExampleHealthCheck` class demonstrates the layout of a health check.</span></span> <span data-ttu-id="32395-153">Логика проверок работоспособности помещается в метод `CheckHealthAsync`.</span><span class="sxs-lookup"><span data-stu-id="32395-153">The health checks logic is placed in the `CheckHealthAsync` method.</span></span> <span data-ttu-id="32395-154">В следующем примере для `true` задается фиктивная переменная `healthCheckResultHealthy`.</span><span class="sxs-lookup"><span data-stu-id="32395-154">The following example sets a dummy variable, `healthCheckResultHealthy`, to `true`.</span></span> <span data-ttu-id="32395-155">Если для `healthCheckResultHealthy` задано значение `false`, возвращается состояние [HealthCheckResult.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult.Unhealthy*).</span><span class="sxs-lookup"><span data-stu-id="32395-155">If the value of `healthCheckResultHealthy` is set to `false`, the [HealthCheckResult.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult.Unhealthy*) status is returned.</span></span>

```csharp
public class ExampleHealthCheck : IHealthCheck
{
    public Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        var healthCheckResultHealthy = true;

        if (healthCheckResultHealthy)
        {
            return Task.FromResult(
                HealthCheckResult.Healthy("A healthy result."));
        }

        return Task.FromResult(
            HealthCheckResult.Unhealthy("An unhealthy result."));
    }
}
```

## <a name="register-health-check-services"></a><span data-ttu-id="32395-156">Зарегистрируйте службы проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-156">Register health check services</span></span>

<span data-ttu-id="32395-157">Тип `ExampleHealthCheck` будет добавлен к службам проверки работоспособности с помощью <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> в `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="32395-157">The `ExampleHealthCheck` type is added to health check services with <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> in `Startup.ConfigureServices`:</span></span>

```csharp
services.AddHealthChecks()
    .AddCheck<ExampleHealthCheck>("example_health_check");
```

<span data-ttu-id="32395-158">Перегрузка <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*>, показанная в следующем примере, устанавливает состояние сбоя (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) для отчета в ситуации, когда проверка работоспособности сообщает о сбое.</span><span class="sxs-lookup"><span data-stu-id="32395-158">The <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> overload shown in the following example sets the failure status (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) to report when the health check reports a failure.</span></span> <span data-ttu-id="32395-159">Если состояние сбоя имеет значение `null` (по умолчанию), сообщается [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus).</span><span class="sxs-lookup"><span data-stu-id="32395-159">If the failure status is set to `null` (default), [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) is reported.</span></span> <span data-ttu-id="32395-160">Эта перегрузка — полезный сценарий для авторов библиотек, где состояние сбоя от библиотеки особо учитывается приложением при сбое проверки работоспособности, если реализация проверки работоспособности учитывает этот параметр.</span><span class="sxs-lookup"><span data-stu-id="32395-160">This overload is a useful scenario for library authors, where the failure status indicated by the library is enforced by the app when a health check failure occurs if the health check implementation honors the setting.</span></span>

<span data-ttu-id="32395-161">*Теги* можно использовать для фильтрации проверок работоспособности (описаны далее в разделе [Фильтрация проверок работоспособности](#filter-health-checks)).</span><span class="sxs-lookup"><span data-stu-id="32395-161">*Tags* can be used to filter health checks (described further in the [Filter health checks](#filter-health-checks) section).</span></span>

```csharp
services.AddHealthChecks()
    .AddCheck<ExampleHealthCheck>(
        "example_health_check",
        failureStatus: HealthStatus.Degraded,
        tags: new[] { "example" });
```

<span data-ttu-id="32395-162"><xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> также может выполнять лямбда-функции.</span><span class="sxs-lookup"><span data-stu-id="32395-162"><xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> can also execute a lambda function.</span></span> <span data-ttu-id="32395-163">В следующем примере имя проверки работоспособности указано как `Example`; проверка всегда возвращает работоспособное состояние:</span><span class="sxs-lookup"><span data-stu-id="32395-163">In the following example, the health check name is specified as `Example` and the check always returns a healthy state:</span></span>

```csharp
services.AddHealthChecks()
    .AddCheck("Example", () =>
        HealthCheckResult.Healthy("Example is OK!"), tags: new[] { "example" });
```

<span data-ttu-id="32395-164">Вызовите <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddTypeActivatedCheck*>, чтобы передать аргументы реализации проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-164">Call <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddTypeActivatedCheck*> to pass arguments to a health check implementation.</span></span> <span data-ttu-id="32395-165">В следующем примере `TestHealthCheckWithArgs` принимает целое число и строку для использования при вызове <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck.CheckHealthAsync*>:</span><span class="sxs-lookup"><span data-stu-id="32395-165">In the following example, `TestHealthCheckWithArgs` accepts an integer and a string for use when <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck.CheckHealthAsync*> is called:</span></span>

```csharp
private class TestHealthCheckWithArgs : IHealthCheck
{
    public TestHealthCheckWithArgs(int i, string s)
    {
        I = i;
        S = s;
    }

    public int I { get; set; }

    public string S { get; set; }

    public Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, 
        CancellationToken cancellationToken = default)
    {
        ...
    }
}
```

<span data-ttu-id="32395-166">`TestHealthCheckWithArgs` регистрируется путем вызова `AddTypeActivatedCheck` с целым числом и строкой, передаваемой в реализацию:</span><span class="sxs-lookup"><span data-stu-id="32395-166">`TestHealthCheckWithArgs` is registered by calling `AddTypeActivatedCheck` with the integer and string passed to the implementation:</span></span>

```csharp
services.AddHealthChecks()
    .AddTypeActivatedCheck<TestHealthCheckWithArgs>(
        "test", 
        failureStatus: HealthStatus.Degraded, 
        tags: new[] { "example" }, 
        args: new object[] { 5, "string" });
```

## <a name="use-health-checks-routing"></a><span data-ttu-id="32395-167">Использование маршрутизации при проверках работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-167">Use Health Checks Routing</span></span>

<span data-ttu-id="32395-168">В `Startup.Configure` вызовите `MapHealthChecks` для построителя конечной точки с URL-адресом конечной точки или относительным путем:</span><span class="sxs-lookup"><span data-stu-id="32395-168">In `Startup.Configure`, call `MapHealthChecks` on the endpoint builder with the endpoint URL or relative path:</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health");
});
```

### <a name="require-host"></a><span data-ttu-id="32395-169">RequireHost</span><span class="sxs-lookup"><span data-stu-id="32395-169">Require host</span></span>

<span data-ttu-id="32395-170">Вызовите `RequireHost`, чтобы указать один или несколько разрешенных узлов для конечной точки проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-170">Call `RequireHost` to specify one or more permitted hosts for the health check endpoint.</span></span> <span data-ttu-id="32395-171">Узлы должны быть указаны в Юникоде, а не Punycode, и могут включать порт.</span><span class="sxs-lookup"><span data-stu-id="32395-171">Hosts should be Unicode rather than punycode and may include a port.</span></span> <span data-ttu-id="32395-172">Если коллекция не указана, допускается любой узел.</span><span class="sxs-lookup"><span data-stu-id="32395-172">If a collection isn't supplied, any host is accepted.</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health").RequireHost("www.contoso.com:5001");
});
```

<span data-ttu-id="32395-173">Дополнительные сведения см. в разделе [Фильтр по портам](#filter-by-port).</span><span class="sxs-lookup"><span data-stu-id="32395-173">For more information, see the [Filter by port](#filter-by-port) section.</span></span>

### <a name="require-authorization"></a><span data-ttu-id="32395-174">RequireAuthorization</span><span class="sxs-lookup"><span data-stu-id="32395-174">Require authorization</span></span>

<span data-ttu-id="32395-175">Вызовите `RequireAuthorization` для запуска ПО промежуточного слоя для авторизации в конечной точке запроса проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-175">Call `RequireAuthorization` to run Authorization Middleware on the health check request endpoint.</span></span> <span data-ttu-id="32395-176">Перегрузка `RequireAuthorization` принимает одну или несколько политик авторизации.</span><span class="sxs-lookup"><span data-stu-id="32395-176">A `RequireAuthorization` overload accepts one or more authorization policies.</span></span> <span data-ttu-id="32395-177">Если политика не указана, используется политика авторизации по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="32395-177">If a policy isn't provided, the default authorization policy is used.</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health").RequireAuthorization();
});
```

### <a name="enable-cross-origin-requests-cors"></a><span data-ttu-id="32395-178">Включение запросов о происхождении (CORS)</span><span class="sxs-lookup"><span data-stu-id="32395-178">Enable Cross-Origin Requests (CORS)</span></span>

<span data-ttu-id="32395-179">Хотя выполнение проверок работоспособности вручную из браузера не является распространенным сценарием использования, ПО промежуточного слоя CORS можно включить, вызвав `RequireCors` в конечных точках проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-179">Although performing health checks manually from a browser isn't a common use scenario, CORS Middleware can be enabled by calling `RequireCors` on health checks endpoints.</span></span> <span data-ttu-id="32395-180">Перегрузка `RequireCors` принимает делегат построителя политики CORS (`CorsPolicyBuilder`) или имя политики.</span><span class="sxs-lookup"><span data-stu-id="32395-180">A `RequireCors` overload accepts a CORS policy builder delegate (`CorsPolicyBuilder`) or a policy name.</span></span> <span data-ttu-id="32395-181">Если политика не указана, используется политика CORS по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="32395-181">If a policy isn't provided, the default CORS policy is used.</span></span> <span data-ttu-id="32395-182">Для получения дополнительной информации см. <xref:security/cors>.</span><span class="sxs-lookup"><span data-stu-id="32395-182">For more information, see <xref:security/cors>.</span></span>

## <a name="health-check-options"></a><span data-ttu-id="32395-183">Варианты проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-183">Health check options</span></span>

<span data-ttu-id="32395-184"><xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions> предоставляет возможность настройки поведения для проверки работоспособности:</span><span class="sxs-lookup"><span data-stu-id="32395-184"><xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions> provide an opportunity to customize health check behavior:</span></span>

* [<span data-ttu-id="32395-185">Фильтрация проверок работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-185">Filter health checks</span></span>](#filter-health-checks)
* [<span data-ttu-id="32395-186">Настройка кода состояния HTTP</span><span class="sxs-lookup"><span data-stu-id="32395-186">Customize the HTTP status code</span></span>](#customize-the-http-status-code)
* [<span data-ttu-id="32395-187">Подавление заголовков кэша</span><span class="sxs-lookup"><span data-stu-id="32395-187">Suppress cache headers</span></span>](#suppress-cache-headers)
* [<span data-ttu-id="32395-188">Настройка выходных данных</span><span class="sxs-lookup"><span data-stu-id="32395-188">Customize output</span></span>](#customize-output)

### <a name="filter-health-checks"></a><span data-ttu-id="32395-189">Фильтрация проверок работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-189">Filter health checks</span></span>

<span data-ttu-id="32395-190">По умолчанию ПО промежуточного слоя для проверки работоспособности выполняет все зарегистрированные проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-190">By default, Health Checks Middleware runs all registered health checks.</span></span> <span data-ttu-id="32395-191">Чтобы выполнить подмножество проверок работоспособности, укажите функцию, которая возвращает логическое значение через параметр <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate>.</span><span class="sxs-lookup"><span data-stu-id="32395-191">To run a subset of health checks, provide a function that returns a boolean to the <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate> option.</span></span> <span data-ttu-id="32395-192">В следующем примере проверка работоспособности `Bar` отфильтровывается по тегу (`bar_tag`) в условном операторе функции, где `true` возвращается только в том случае, если свойство проверки работоспособности <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.Tags> соответствует `foo_tag` или `baz_tag`:</span><span class="sxs-lookup"><span data-stu-id="32395-192">In the following example, the `Bar` health check is filtered out by its tag (`bar_tag`) in the function's conditional statement, where `true` is only returned if the health check's <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.Tags> property matches `foo_tag` or `baz_tag`:</span></span>

<span data-ttu-id="32395-193">В `Startup.ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="32395-193">In `Startup.ConfigureServices`:</span></span>

```csharp
services.AddHealthChecks()
    .AddCheck("Foo", () =>
        HealthCheckResult.Healthy("Foo is OK!"), tags: new[] { "foo_tag" })
    .AddCheck("Bar", () =>
        HealthCheckResult.Unhealthy("Bar is unhealthy!"), tags: new[] { "bar_tag" })
    .AddCheck("Baz", () =>
        HealthCheckResult.Healthy("Baz is OK!"), tags: new[] { "baz_tag" });
```

<span data-ttu-id="32395-194">В `Startup.Configure``Predicate` определяет, что проверка работоспособности Bar не выполняется.</span><span class="sxs-lookup"><span data-stu-id="32395-194">In `Startup.Configure`, the `Predicate` filters out the 'Bar' health check.</span></span> <span data-ttu-id="32395-195">Выполняется только проверка работоспособности Foo и Baz:</span><span class="sxs-lookup"><span data-stu-id="32395-195">Only Foo and Baz execute.:</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health", new HealthCheckOptions()
    {
        Predicate = (check) => check.Tags.Contains("foo_tag") ||
            check.Tags.Contains("baz_tag")
    });
});
```

### <a name="customize-the-http-status-code"></a><span data-ttu-id="32395-196">Настройка кода состояния HTTP</span><span class="sxs-lookup"><span data-stu-id="32395-196">Customize the HTTP status code</span></span>

<span data-ttu-id="32395-197">Используйте <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResultStatusCodes> для настройки сопоставления состояний работоспособности и кодов состояния HTTP.</span><span class="sxs-lookup"><span data-stu-id="32395-197">Use <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResultStatusCodes> to customize the mapping of health status to HTTP status codes.</span></span> <span data-ttu-id="32395-198">Следующие назначения <xref:Microsoft.AspNetCore.Http.StatusCodes> являются значениями по умолчанию, используемыми ПО промежуточного слоя.</span><span class="sxs-lookup"><span data-stu-id="32395-198">The following <xref:Microsoft.AspNetCore.Http.StatusCodes> assignments are the default values used by the middleware.</span></span> <span data-ttu-id="32395-199">Измените значения кодов состояния в соответствии с требованиями.</span><span class="sxs-lookup"><span data-stu-id="32395-199">Change the status code values to meet your requirements.</span></span>

<span data-ttu-id="32395-200">В `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="32395-200">In `Startup.Configure`:</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health", new HealthCheckOptions()
    {
        ResultStatusCodes =
        {
            [HealthStatus.Healthy] = StatusCodes.Status200OK,
            [HealthStatus.Degraded] = StatusCodes.Status200OK,
            [HealthStatus.Unhealthy] = StatusCodes.Status503ServiceUnavailable
        }
    });
});
```

### <a name="suppress-cache-headers"></a><span data-ttu-id="32395-201">Подавление заголовков кэша</span><span class="sxs-lookup"><span data-stu-id="32395-201">Suppress cache headers</span></span>

<span data-ttu-id="32395-202"><xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.AllowCachingResponses> определяет, добавляет ли ПО промежуточного слоя HTTP-заголовки в ответ проверки, чтобы предотвратить кэширование ответов.</span><span class="sxs-lookup"><span data-stu-id="32395-202"><xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.AllowCachingResponses> controls whether the Health Checks Middleware adds HTTP headers to a probe response to prevent response caching.</span></span> <span data-ttu-id="32395-203">Если значение равно `false` (по умолчанию), ПО промежуточного слоя задает или переопределяет заголовки `Cache-Control`, `Expires` и `Pragma`, чтобы предотвратить кэширование ответов.</span><span class="sxs-lookup"><span data-stu-id="32395-203">If the value is `false` (default), the middleware sets or overrides the `Cache-Control`, `Expires`, and `Pragma` headers to prevent response caching.</span></span> <span data-ttu-id="32395-204">Если значение равно `true`, ПО промежуточного слоя не изменяет заголовки кэша в ответе.</span><span class="sxs-lookup"><span data-stu-id="32395-204">If the value is `true`, the middleware doesn't modify the cache headers of the response.</span></span>

<span data-ttu-id="32395-205">В `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="32395-205">In `Startup.Configure`:</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health", new HealthCheckOptions()
    {
        AllowCachingResponses = false
    });
});
```

### <a name="customize-output"></a><span data-ttu-id="32395-206">Настройка выходных данных</span><span class="sxs-lookup"><span data-stu-id="32395-206">Customize output</span></span>

<span data-ttu-id="32395-207">В `Startup.Configure` задайте для параметра [HealthCheckOptions.ResponseWriter](xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResponseWriter) делегат для записи ответа:</span><span class="sxs-lookup"><span data-stu-id="32395-207">In `Startup.Configure`, set the [HealthCheckOptions.ResponseWriter](xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResponseWriter) option to a delegate for writing the response:</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health", new HealthCheckOptions()
    {
        ResponseWriter = WriteResponse
    });
});
```

<span data-ttu-id="32395-208">Делегат по умолчанию записывает минимальный ответ в формате открытого текста со строковым значением [HealthReport.Status](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthReport.Status).</span><span class="sxs-lookup"><span data-stu-id="32395-208">The default delegate writes a minimal plaintext response with the string value of [HealthReport.Status](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthReport.Status).</span></span> <span data-ttu-id="32395-209">Следующие пользовательские делегаты выводят настраиваемый ответ JSON.</span><span class="sxs-lookup"><span data-stu-id="32395-209">The following custom delegates output a custom JSON response.</span></span>

<span data-ttu-id="32395-210">В первом примере из примера приложения показано, как использовать <xref:System.Text.Json?displayProperty=fullName>:</span><span class="sxs-lookup"><span data-stu-id="32395-210">The first example from the sample app demonstrates how to use <xref:System.Text.Json?displayProperty=fullName>:</span></span>

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_WriteResponse_SystemTextJson)]

<span data-ttu-id="32395-211">Во втором примере показано, как использовать [Newtonsoft. JSON](https://www.nuget.org/packages/Newtonsoft.Json/):</span><span class="sxs-lookup"><span data-stu-id="32395-211">The second example demonstrates how to use [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/):</span></span>

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_WriteResponse_NewtonSoftJson)]

<span data-ttu-id="32395-212">В примере приложения закомментируйте [директиву препроцессора](xref:index#preprocessor-directives-in-sample-code) `SYSTEM_TEXT_JSON` в *CustomWriterStartup.cs*, чтобы включить версию `Newtonsoft.Json` `WriteResponse`.</span><span class="sxs-lookup"><span data-stu-id="32395-212">In the sample app, comment out the `SYSTEM_TEXT_JSON` [preprocessor directive](xref:index#preprocessor-directives-in-sample-code) in *CustomWriterStartup.cs* to enable the `Newtonsoft.Json` version of `WriteResponse`.</span></span>

<span data-ttu-id="32395-213">API проверки работоспособности не предоставляет встроенной поддержки сложных форматов возврата JSON, так как формат зависит от выбранной системы мониторинга.</span><span class="sxs-lookup"><span data-stu-id="32395-213">The health checks API doesn't provide built-in support for complex JSON return formats because the format is specific to your choice of monitoring system.</span></span> <span data-ttu-id="32395-214">При необходимости настройте ответ в предыдущих примерах.</span><span class="sxs-lookup"><span data-stu-id="32395-214">Customize the response in the preceding examples as needed.</span></span> <span data-ttu-id="32395-215">Дополнительные сведения о сериализации JSON с `System.Text.Json` см. в статье о [сериализации и десериализации JSON в .NET](/dotnet/standard/serialization/system-text-json-how-to).</span><span class="sxs-lookup"><span data-stu-id="32395-215">For more information on JSON serialization with `System.Text.Json`, see [How to serialize and deserialize JSON in .NET](/dotnet/standard/serialization/system-text-json-how-to).</span></span>

## <a name="database-probe"></a><span data-ttu-id="32395-216">Проверка базы данных</span><span class="sxs-lookup"><span data-stu-id="32395-216">Database probe</span></span>

<span data-ttu-id="32395-217">Проверка работоспособности может задать запрос для выполнения в качестве логического теста для указания того, отвечает ли база данных как обычно.</span><span class="sxs-lookup"><span data-stu-id="32395-217">A health check can specify a database query to run as a boolean test to indicate if the database is responding normally.</span></span>

<span data-ttu-id="32395-218">Пример приложения использует [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks), библиотеку проверки работоспособности для приложений ASP.NET Core, чтобы проверить работоспособность базы данных SQL Server.</span><span class="sxs-lookup"><span data-stu-id="32395-218">The sample app uses [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks), a health check library for ASP.NET Core apps, to perform a health check on a SQL Server database.</span></span> <span data-ttu-id="32395-219">`AspNetCore.Diagnostics.HealthChecks` выполняет запрос `SELECT 1` к базе данных, чтобы подтвердить, что подключение к базе данных находится в работоспособном состоянии.</span><span class="sxs-lookup"><span data-stu-id="32395-219">`AspNetCore.Diagnostics.HealthChecks` executes a `SELECT 1` query against the database to confirm the connection to the database is healthy.</span></span>

> [!WARNING]
> <span data-ttu-id="32395-220">При проверке подключения к базе данных с помощью запроса выберите запрос, возвращающий результат быстро.</span><span class="sxs-lookup"><span data-stu-id="32395-220">When checking a database connection with a query, choose a query that returns quickly.</span></span> <span data-ttu-id="32395-221">Метод запроса связан с риском перегрузки базы данных и снижения ее производительности.</span><span class="sxs-lookup"><span data-stu-id="32395-221">The query approach runs the risk of overloading the database and degrading its performance.</span></span> <span data-ttu-id="32395-222">В большинстве случаев тестовый запрос не является обязательным.</span><span class="sxs-lookup"><span data-stu-id="32395-222">In most cases, running a test query isn't necessary.</span></span> <span data-ttu-id="32395-223">Достаточно успешного подключения к базе данных.</span><span class="sxs-lookup"><span data-stu-id="32395-223">Merely making a successful connection to the database is sufficient.</span></span> <span data-ttu-id="32395-224">Если вам понадобится выполнить запрос, выберите простой запрос SELECT, например `SELECT 1`.</span><span class="sxs-lookup"><span data-stu-id="32395-224">If you find it necessary to run a query, choose a simple SELECT query, such as `SELECT 1`.</span></span>

<span data-ttu-id="32395-225">Добавьте ссылку на пакет [AspNetCore.HealthChecks.SqlServer](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/).</span><span class="sxs-lookup"><span data-stu-id="32395-225">Include a package reference to [AspNetCore.HealthChecks.SqlServer](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/).</span></span>

<span data-ttu-id="32395-226">Укажите допустимую строку подключения к базе данных в файле примера приложения *appsettings.json*.</span><span class="sxs-lookup"><span data-stu-id="32395-226">Supply a valid database connection string in the *appsettings.json* file of the sample app.</span></span> <span data-ttu-id="32395-227">Приложение использует базу данных SQL Server с именем `HealthCheckSample`:</span><span class="sxs-lookup"><span data-stu-id="32395-227">The app uses a SQL Server database named `HealthCheckSample`:</span></span>

[!code-json[](health-checks/samples/3.x/HealthChecksSample/appsettings.json?highlight=3)]

<span data-ttu-id="32395-228">Зарегистрируйте службы проверки работоспособности при помощи <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> в `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="32395-228">Register health check services with <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="32395-229">Пример приложения вызывает метод `AddSqlServer` со строкой подключения базы данных (*DbHealthStartup.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-229">The sample app calls the `AddSqlServer` method with the database's connection string (*DbHealthStartup.cs*):</span></span>

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/DbHealthStartup.cs?name=snippet_ConfigureServices)]

<span data-ttu-id="32395-230">Конечная точка проверки работоспособности создается путем вызова `MapHealthChecks` в `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="32395-230">A health check endpoint is created by calling `MapHealthChecks` in `Startup.Configure`:</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health");
}
```

<span data-ttu-id="32395-231">Чтобы запустить сценарий проверки базы данных с помощью примера приложения, выполните следующую команду из папки проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-231">To run the database probe scenario using the sample app, execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario db
```

> [!NOTE]
> <span data-ttu-id="32395-232">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) не поддерживается корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="32395-232">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) isn't maintained or supported by Microsoft.</span></span>

## <a name="entity-framework-core-dbcontext-probe"></a><span data-ttu-id="32395-233">Проверка DbContext в Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="32395-233">Entity Framework Core DbContext probe</span></span>

<span data-ttu-id="32395-234">Эта проверка `DbContext` подтверждает, что приложение может взаимодействовать с базой данных, настроенной для EF Core `DbContext`.</span><span class="sxs-lookup"><span data-stu-id="32395-234">The `DbContext` check confirms that the app can communicate with the database configured for an EF Core `DbContext`.</span></span> <span data-ttu-id="32395-235">Проверка `DbContext` поддерживается в приложениях, которые:</span><span class="sxs-lookup"><span data-stu-id="32395-235">The `DbContext` check is supported in apps that:</span></span>

* <span data-ttu-id="32395-236">используют [Entity Framework (EF) Core](/ef/core/);</span><span class="sxs-lookup"><span data-stu-id="32395-236">Use [Entity Framework (EF) Core](/ef/core/).</span></span>
* <span data-ttu-id="32395-237">содержат ссылку на пакет [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/).</span><span class="sxs-lookup"><span data-stu-id="32395-237">Include a package reference to [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/).</span></span>

<span data-ttu-id="32395-238">`AddDbContextCheck<TContext>` регистрирует проверку работоспособности для `DbContext`.</span><span class="sxs-lookup"><span data-stu-id="32395-238">`AddDbContextCheck<TContext>` registers a health check for a `DbContext`.</span></span> <span data-ttu-id="32395-239">`DbContext` передается методу в качестве `TContext`.</span><span class="sxs-lookup"><span data-stu-id="32395-239">The `DbContext` is supplied as the `TContext` to the method.</span></span> <span data-ttu-id="32395-240">Доступна перегрузка, позволяющая настроить состояние ошибки, теги и запрос тестирования.</span><span class="sxs-lookup"><span data-stu-id="32395-240">An overload is available to configure the failure status, tags, and a custom test query.</span></span>

<span data-ttu-id="32395-241">По умолчанию:</span><span class="sxs-lookup"><span data-stu-id="32395-241">By default:</span></span>

* <span data-ttu-id="32395-242">`DbContextHealthCheck` вызывает метод EF Core `CanConnectAsync`.</span><span class="sxs-lookup"><span data-stu-id="32395-242">The `DbContextHealthCheck` calls EF Core's `CanConnectAsync` method.</span></span> <span data-ttu-id="32395-243">Вы можете указать, какая операция выполняется в том случае, когда проверка работоспособности производится с помощью перегрузок метода `AddDbContextCheck`.</span><span class="sxs-lookup"><span data-stu-id="32395-243">You can customize what operation is run when checking health using `AddDbContextCheck` method overloads.</span></span>
* <span data-ttu-id="32395-244">Имя проверки работоспособности — это имя типа `TContext`.</span><span class="sxs-lookup"><span data-stu-id="32395-244">The name of the health check is the name of the `TContext` type.</span></span>

<span data-ttu-id="32395-245">В примере приложения `AppDbContext` предоставляется для `AddDbContextCheck` и регистрируется в качестве службы в `Startup.ConfigureServices` (*DbContextHealthStartup.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-245">In the sample app, `AppDbContext` is provided to `AddDbContextCheck` and registered as a service in `Startup.ConfigureServices` (*DbContextHealthStartup.cs*):</span></span>

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/DbContextHealthStartup.cs?name=snippet_ConfigureServices)]

<span data-ttu-id="32395-246">Конечная точка проверки работоспособности создается путем вызова `MapHealthChecks` в `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="32395-246">A health check endpoint is created by calling `MapHealthChecks` in `Startup.Configure`:</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health");
}
```

<span data-ttu-id="32395-247">Прежде чем запускать сценарий проверки `DbContext` с помощью примера приложения, убедитесь, что база данных, указанная в строке подключения, не существует в экземпляре SQL Server.</span><span class="sxs-lookup"><span data-stu-id="32395-247">To run the `DbContext` probe scenario using the sample app, confirm that the database specified by the connection string doesn't exist in the SQL Server instance.</span></span> <span data-ttu-id="32395-248">Если база данных существует, удалите ее.</span><span class="sxs-lookup"><span data-stu-id="32395-248">If the database exists, delete it.</span></span>

<span data-ttu-id="32395-249">Выполните следующую команду в папке проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-249">Execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario dbcontext
```

<span data-ttu-id="32395-250">После запуска приложения проверьте состояние работоспособности, выполнив запрос к конечной точке `/health` в браузере.</span><span class="sxs-lookup"><span data-stu-id="32395-250">After the app is running, check the health status by making a request to the `/health` endpoint in a browser.</span></span> <span data-ttu-id="32395-251">Базы данных и `AppDbContext` не существуют, поэтому приложение дает следующий ответ:</span><span class="sxs-lookup"><span data-stu-id="32395-251">The database and `AppDbContext` don't exist, so app provides the following response:</span></span>

```
Unhealthy
```

<span data-ttu-id="32395-252">Заставьте пример приложения создать базу данных.</span><span class="sxs-lookup"><span data-stu-id="32395-252">Trigger the sample app to create the database.</span></span> <span data-ttu-id="32395-253">Сделайте запрос к `/createdatabase`.</span><span class="sxs-lookup"><span data-stu-id="32395-253">Make a request to `/createdatabase`.</span></span> <span data-ttu-id="32395-254">Приложение выдает ответ:</span><span class="sxs-lookup"><span data-stu-id="32395-254">The app responds:</span></span>

```
Creating the database...
Done!
Navigate to /health to see the health status.
```

<span data-ttu-id="32395-255">Сделайте запрос к конечной точке `/health`.</span><span class="sxs-lookup"><span data-stu-id="32395-255">Make a request to the `/health` endpoint.</span></span> <span data-ttu-id="32395-256">База данных и контекст существуют, поэтому приложение отвечает:</span><span class="sxs-lookup"><span data-stu-id="32395-256">The database and context exist, so app responds:</span></span>

```
Healthy
```

<span data-ttu-id="32395-257">Заставьте пример приложения удалить базу данных.</span><span class="sxs-lookup"><span data-stu-id="32395-257">Trigger the sample app to delete the database.</span></span> <span data-ttu-id="32395-258">Сделайте запрос к `/deletedatabase`.</span><span class="sxs-lookup"><span data-stu-id="32395-258">Make a request to `/deletedatabase`.</span></span> <span data-ttu-id="32395-259">Приложение выдает ответ:</span><span class="sxs-lookup"><span data-stu-id="32395-259">The app responds:</span></span>

```
Deleting the database...
Done!
Navigate to /health to see the health status.
```

<span data-ttu-id="32395-260">Сделайте запрос к конечной точке `/health`.</span><span class="sxs-lookup"><span data-stu-id="32395-260">Make a request to the `/health` endpoint.</span></span> <span data-ttu-id="32395-261">Приложение предоставляет ответ о неработоспособности:</span><span class="sxs-lookup"><span data-stu-id="32395-261">The app provides an unhealthy response:</span></span>

```
Unhealthy
```

## <a name="separate-readiness-and-liveness-probes"></a><span data-ttu-id="32395-262">Разделение проверок готовности и активности</span><span class="sxs-lookup"><span data-stu-id="32395-262">Separate readiness and liveness probes</span></span>

<span data-ttu-id="32395-263">В некоторых сценариях размещения используются пары проверок работоспособности, отличающие два состояния приложения:</span><span class="sxs-lookup"><span data-stu-id="32395-263">In some hosting scenarios, a pair of health checks are used that distinguish two app states:</span></span>

* <span data-ttu-id="32395-264">Приложение работает, но еще не готово к получению запросов.</span><span class="sxs-lookup"><span data-stu-id="32395-264">The app is functioning but not yet ready to receive requests.</span></span> <span data-ttu-id="32395-265">Это состояние *готовности* приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-265">This state is the app's *readiness*.</span></span>
* <span data-ttu-id="32395-266">Приложение работает и отвечает на запросы.</span><span class="sxs-lookup"><span data-stu-id="32395-266">The app is functioning and responding to requests.</span></span> <span data-ttu-id="32395-267">Это состояние *жизнеспособности* приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-267">This state is the app's *liveness*.</span></span>

<span data-ttu-id="32395-268">Проверка готовности обычно выполняет ряд проверок, чтобы определить, все ли подсистемы и ресурсы приложения доступны; она более обширна и требует много времени.</span><span class="sxs-lookup"><span data-stu-id="32395-268">The readiness check usually performs a more extensive and time-consuming set of checks to determine if all of the app's subsystems and resources are available.</span></span> <span data-ttu-id="32395-269">Проверка жизнеспособности лишь быстро определяет, доступно ли приложение для обработки запросов.</span><span class="sxs-lookup"><span data-stu-id="32395-269">A liveness check merely performs a quick check to determine if the app is available to process requests.</span></span> <span data-ttu-id="32395-270">По прохождении проверки готовности приложения нет необходимости создавать чрезмерную нагрузку на приложение с дорогостоящим набором проверок готовности&mdash;дальнейшие проверки требуют лишь проверки жизнеспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-270">After the app passes its readiness check, there's no need to burden the app further with the expensive set of readiness checks&mdash;further checks only require checking for liveness.</span></span>

<span data-ttu-id="32395-271">Пример приложения содержит проверку работоспособности, которая сообщает о завершении длительной задачи запуска [размещенной службы](xref:fundamentals/host/hosted-services).</span><span class="sxs-lookup"><span data-stu-id="32395-271">The sample app contains a health check to report the completion of long-running startup task in a [Hosted Service](xref:fundamentals/host/hosted-services).</span></span> <span data-ttu-id="32395-272">`StartupHostedServiceHealthCheck` предоставляет свойство, `StartupTaskCompleted`, которое размещенная служба может задать как `true` после завершения длительной задачи (*StartupHostedServiceHealthCheck.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-272">The `StartupHostedServiceHealthCheck` exposes a property, `StartupTaskCompleted`, that the hosted service can set to `true` when its long-running task is finished (*StartupHostedServiceHealthCheck.cs*):</span></span>

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/StartupHostedServiceHealthCheck.cs?name=snippet1&highlight=7-11)]

<span data-ttu-id="32395-273">Длительная фоновая задача запущена [размещенной службой](xref:fundamentals/host/hosted-services) (*Services/StartupHostedService*).</span><span class="sxs-lookup"><span data-stu-id="32395-273">The long-running background task is started by a [Hosted Service](xref:fundamentals/host/hosted-services) (*Services/StartupHostedService*).</span></span> <span data-ttu-id="32395-274">По завершении задачи `StartupHostedServiceHealthCheck.StartupTaskCompleted` получает значение `true`:</span><span class="sxs-lookup"><span data-stu-id="32395-274">At the conclusion of the task, `StartupHostedServiceHealthCheck.StartupTaskCompleted` is set to `true`:</span></span>

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/Services/StartupHostedService.cs?name=snippet1&highlight=18-20)]

<span data-ttu-id="32395-275">Проверка работоспособности регистрируется через <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> в `Startup.ConfigureServices` вместе с размещенной службой.</span><span class="sxs-lookup"><span data-stu-id="32395-275">The health check is registered with <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> in `Startup.ConfigureServices` along with the hosted service.</span></span> <span data-ttu-id="32395-276">Поскольку размещенной службе необходимо задать свойство проверки работоспособности, проверка работоспособности также регистрируется в контейнере службы (*LivenessProbeStartup.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-276">Because the hosted service must set the property on the health check, the health check is also registered in the service container (*LivenessProbeStartup.cs*):</span></span>

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/LivenessProbeStartup.cs?name=snippet_ConfigureServices)]

<span data-ttu-id="32395-277">Конечная точка проверки работоспособности создается путем вызова `MapHealthChecks` в `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-277">A health check endpoint is created by calling `MapHealthChecks` in `Startup.Configure`.</span></span> <span data-ttu-id="32395-278">В примере приложения конечные точки проверки работоспособности создаются в следующих расположениях:</span><span class="sxs-lookup"><span data-stu-id="32395-278">In the sample app, the health check endpoints are created at:</span></span>

* <span data-ttu-id="32395-279">В `/health/ready` для проверки готовности.</span><span class="sxs-lookup"><span data-stu-id="32395-279">`/health/ready` for the readiness check.</span></span> <span data-ttu-id="32395-280">Проверка готовности фильтрует проверки работоспособности по тегу `ready`.</span><span class="sxs-lookup"><span data-stu-id="32395-280">The readiness check filters health checks to the health check with the `ready` tag.</span></span>
* <span data-ttu-id="32395-281">В `/health/live` для проверки жизнеспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-281">`/health/live` for the liveness check.</span></span> <span data-ttu-id="32395-282">Проверка жизнеспособности отфильтровывает `StartupHostedServiceHealthCheck`, возвращая `false` в [HealthCheckOptions.Predicate](xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate) (дополнительные сведения см. в разделе [Фильтрация проверок работоспособности](#filter-health-checks)).</span><span class="sxs-lookup"><span data-stu-id="32395-282">The liveness check filters out the `StartupHostedServiceHealthCheck` by returning `false` in the [HealthCheckOptions.Predicate](xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate) (for more information, see [Filter health checks](#filter-health-checks))</span></span>

<span data-ttu-id="32395-283">В следующем примере кода происходит следующее:</span><span class="sxs-lookup"><span data-stu-id="32395-283">In the following example code:</span></span>

* <span data-ttu-id="32395-284">Проверка готовности использует все зарегистрированные проверки с тегом ready.</span><span class="sxs-lookup"><span data-stu-id="32395-284">The readiness check uses all registered checks with the 'ready' tag.</span></span>
* <span data-ttu-id="32395-285">`Predicate` исключает все проверки и возвращает 200 — OK.</span><span class="sxs-lookup"><span data-stu-id="32395-285">The `Predicate` excludes all checks and return a 200-Ok.</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health/ready", new HealthCheckOptions()
    {
        Predicate = (check) => check.Tags.Contains("ready"),
    });

    endpoints.MapHealthChecks("/health/live", new HealthCheckOptions()
    {
        Predicate = (_) => false
    });
}
```

<span data-ttu-id="32395-286">Чтобы запустить сценарий конфигурации проверок готовности и жизнеспособности с помощью примера приложения, выполните следующую команду из папки проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-286">To run the readiness/liveness configuration scenario using the sample app, execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario liveness
```

<span data-ttu-id="32395-287">В браузере посетите `/health/ready` несколько раз, пока не пройдет 15 секунд.</span><span class="sxs-lookup"><span data-stu-id="32395-287">In a browser, visit `/health/ready` several times until 15 seconds have passed.</span></span> <span data-ttu-id="32395-288">Проверка работоспособности будет передавать состояние *Unhealthy* первые 15 секунд.</span><span class="sxs-lookup"><span data-stu-id="32395-288">The health check reports *Unhealthy* for the first 15 seconds.</span></span> <span data-ttu-id="32395-289">По истечении 15 секунд конечная точка передает состояние *Healthy*, что означает завершение длительной задачи размещенной службой.</span><span class="sxs-lookup"><span data-stu-id="32395-289">After 15 seconds, the endpoint reports *Healthy*, which reflects the completion of the long-running task by the hosted service.</span></span>

<span data-ttu-id="32395-290">В этом примере также создается издатель проверки работоспособности (реализация <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>), который выполнит первую проверку готовности с задержкой в две секунды.</span><span class="sxs-lookup"><span data-stu-id="32395-290">This example also creates a Health Check Publisher (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> implementation) that runs the first readiness check with a two second delay.</span></span> <span data-ttu-id="32395-291">Дополнительные сведения см. в разделе [об издателе проверки работоспособности](#health-check-publisher).</span><span class="sxs-lookup"><span data-stu-id="32395-291">For more information, see the [Health Check Publisher](#health-check-publisher) section.</span></span>

### <a name="kubernetes-example"></a><span data-ttu-id="32395-292">Пример для Kubernetes</span><span class="sxs-lookup"><span data-stu-id="32395-292">Kubernetes example</span></span>

<span data-ttu-id="32395-293">Использование отдельных проверок готовности и жизнеспособности полезно в таких средах, как [Kubernetes](https://kubernetes.io/).</span><span class="sxs-lookup"><span data-stu-id="32395-293">Using separate readiness and liveness checks is useful in an environment such as [Kubernetes](https://kubernetes.io/).</span></span> <span data-ttu-id="32395-294">В Kubernetes приложениям может потребоваться долго выполнять задачи во время запуска, прежде чем начать принимать запросы; например, проверить доступность основной базы данных.</span><span class="sxs-lookup"><span data-stu-id="32395-294">In Kubernetes, an app might be required to perform time-consuming startup work before accepting requests, such as a test of the underlying database availability.</span></span> <span data-ttu-id="32395-295">Использование отдельных проверок позволяет оркестратору различать, когда приложение работает, но еще не готово, а когда приложение не удалось запустить.</span><span class="sxs-lookup"><span data-stu-id="32395-295">Using separate checks allows the orchestrator to distinguish whether the app is functioning but not yet ready or if the app has failed to start.</span></span> <span data-ttu-id="32395-296">Дополнительные сведения о проверках готовности и жизнеспособности в Kubernetes см. в разделе [Настройка проверок готовности и жизнеспособности](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) в документации по Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="32395-296">For more information on readiness and liveness probes in Kubernetes, see [Configure Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) in the Kubernetes documentation.</span></span>

<span data-ttu-id="32395-297">В следующем примере показана конфигурация проверки готовности для Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="32395-297">The following example demonstrates a Kubernetes readiness probe configuration:</span></span>

```yml
spec:
  template:
  spec:
    readinessProbe:
      # an http probe
      httpGet:
        path: /health/ready
        port: 80
      # length of time to wait for a pod to initialize
      # after pod startup, before applying health checking
      initialDelaySeconds: 30
      timeoutSeconds: 1
    ports:
      - containerPort: 80
```

## <a name="metric-based-probe-with-a-custom-response-writer"></a><span data-ttu-id="32395-298">Проба на основе метрики с настраиваемым модулем записи ответов</span><span class="sxs-lookup"><span data-stu-id="32395-298">Metric-based probe with a custom response writer</span></span>

<span data-ttu-id="32395-299">Этот пример демонстрирует проверку работоспособности памяти с настраиваемым модулем записи ответов.</span><span class="sxs-lookup"><span data-stu-id="32395-299">The sample app demonstrates a memory health check with a custom response writer.</span></span>

<span data-ttu-id="32395-300">Если приложение использует больше заданного порогового значения памяти (1 ГБ в примере приложения), `MemoryHealthCheck` сообщает о состоянии пониженной функциональности.</span><span class="sxs-lookup"><span data-stu-id="32395-300">`MemoryHealthCheck` reports a degraded status if the app uses more than a given threshold of memory (1 GB in the sample app).</span></span> <span data-ttu-id="32395-301"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> включает сведения от сборщика мусора для приложения (*MemoryHealthCheck.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-301">The <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> includes Garbage Collector (GC) information for the app (*MemoryHealthCheck.cs*):</span></span>

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/MemoryHealthCheck.cs?name=snippet1)]

<span data-ttu-id="32395-302">Зарегистрируйте службы проверки работоспособности при помощи <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> в `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="32395-302">Register health check services with <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="32395-303">Вместо того чтобы включать проверку работоспособности, передав ее в <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*>, `MemoryHealthCheck` регистрируется в качестве службы.</span><span class="sxs-lookup"><span data-stu-id="32395-303">Instead of enabling the health check by passing it to <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*>, the `MemoryHealthCheck` is registered as a service.</span></span> <span data-ttu-id="32395-304">Все зарегистрированные службы <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> доступны в службах проверки работоспособности и ПО промежуточного слоя.</span><span class="sxs-lookup"><span data-stu-id="32395-304">All <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> registered services are available to the health check services and middleware.</span></span> <span data-ttu-id="32395-305">Мы рекомендуем регистрировать службы проверки работоспособности в качестве одноэлементных служб.</span><span class="sxs-lookup"><span data-stu-id="32395-305">We recommend registering health check services as Singleton services.</span></span>

<span data-ttu-id="32395-306">В *CustomWriterStartup.CS* примера приложения выполняется следующее:</span><span class="sxs-lookup"><span data-stu-id="32395-306">In *CustomWriterStartup.cs* of the sample app:</span></span>

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_ConfigureServices&highlight=4)]

<span data-ttu-id="32395-307">Конечная точка проверки работоспособности создается путем вызова `MapHealthChecks` в `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-307">A health check endpoint is created by calling `MapHealthChecks` in `Startup.Configure`.</span></span> <span data-ttu-id="32395-308">Делегат `WriteResponse` предоставляется свойству <Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResponseWriter> для вывода пользовательского ответа JSON при выполнении проверки работоспособности:</span><span class="sxs-lookup"><span data-stu-id="32395-308">A `WriteResponse` delegate is provided to the <Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResponseWriter> property to output a custom JSON response when the health check executes:</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health", new HealthCheckOptions()
    {
        ResponseWriter = WriteResponse
    });
}
```

<span data-ttu-id="32395-309">Делегат `WriteResponse` форматирует `CompositeHealthCheckResult` в объект JSON и выдает выходные данные JSON в качестве ответа проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-309">The `WriteResponse` delegate formats the `CompositeHealthCheckResult` into a JSON object and yields JSON output for the health check response.</span></span> <span data-ttu-id="32395-310">Дополнительные сведения см. в разделе [Настройка выходных данных](#customize-output).</span><span class="sxs-lookup"><span data-stu-id="32395-310">For more information, see the [Customize output](#customize-output) section.</span></span>

<span data-ttu-id="32395-311">Чтобы запустить проверку метрики с настраиваемым модулем записи ответа с помощью примера приложения, выполните следующую команду из папки проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-311">To run the metric-based probe with custom response writer output using the sample app, execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario writer
```

> [!NOTE]
> <span data-ttu-id="32395-312">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) содержит сценарии для проверки работоспособности на основе метрик, включая проверки жизнеспособности для дискового хранилища и максимальных значений.</span><span class="sxs-lookup"><span data-stu-id="32395-312">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) includes metric-based health check scenarios, including disk storage and maximum value liveness checks.</span></span>
>
> <span data-ttu-id="32395-313">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) не поддерживается корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="32395-313">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) isn't maintained or supported by Microsoft.</span></span>

## <a name="filter-by-port"></a><span data-ttu-id="32395-314">Фильтр по портам</span><span class="sxs-lookup"><span data-stu-id="32395-314">Filter by port</span></span>

<span data-ttu-id="32395-315">Вызовите `RequireHost` в `MapHealthChecks` с шаблоном URL-адреса, в котором указан порт для ограничения запросов проверки работоспособности указанным портом.</span><span class="sxs-lookup"><span data-stu-id="32395-315">Call `RequireHost` on `MapHealthChecks` with a URL pattern that specifies a port to restrict health check requests to the port specified.</span></span> <span data-ttu-id="32395-316">Обычно это используется в контейнерной среде с целью предоставления порта для мониторинга служб.</span><span class="sxs-lookup"><span data-stu-id="32395-316">This is typically used in a container environment to expose a port for monitoring services.</span></span>

<span data-ttu-id="32395-317">Пример приложения настраивает порт с помощью [поставщика конфигурации переменных среды](xref:fundamentals/configuration/index#environment-variables-configuration-provider).</span><span class="sxs-lookup"><span data-stu-id="32395-317">The sample app configures the port using the [Environment Variable Configuration Provider](xref:fundamentals/configuration/index#environment-variables-configuration-provider).</span></span> <span data-ttu-id="32395-318">Порт устанавливается в файле *launchSettings.json* и передается поставщику конфигурации с помощью переменной среды.</span><span class="sxs-lookup"><span data-stu-id="32395-318">The port is set in the *launchSettings.json* file and passed to the configuration provider via an environment variable.</span></span> <span data-ttu-id="32395-319">Необходимо также настроить на сервере прослушивание запросов через порт управления.</span><span class="sxs-lookup"><span data-stu-id="32395-319">You must also configure the server to listen to requests on the management port.</span></span>

<span data-ttu-id="32395-320">Чтобы использовать пример приложения для демонстрации настройки портов управления, создайте файл *launchSettings.json* в папке *Properties*.</span><span class="sxs-lookup"><span data-stu-id="32395-320">To use the sample app to demonstrate management port configuration, create the *launchSettings.json* file in a *Properties* folder.</span></span>

<span data-ttu-id="32395-321">Следующий файл примера приложения *Properties/launchSettings.json* отсутствует в файлах проекта примера приложения и должен быть создан вручную:</span><span class="sxs-lookup"><span data-stu-id="32395-321">The following *Properties/launchSettings.json* file in the sample app isn't included in the sample app's project files and must be created manually:</span></span>

```json
{
  "profiles": {
    "SampleApp": {
      "commandName": "Project",
      "commandLineArgs": "",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "ASPNETCORE_URLS": "http://localhost:5000/;http://localhost:5001/",
        "ASPNETCORE_MANAGEMENTPORT": "5001"
      },
      "applicationUrl": "http://localhost:5000/"
    }
  }
}
```

<span data-ttu-id="32395-322">Зарегистрируйте службы проверки работоспособности при помощи <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> в `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="32395-322">Register health check services with <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="32395-323">Создайте конечную точку проверки работоспособности, вызвав `MapHealthChecks` в `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-323">Create a health check endpoint by calling `MapHealthChecks` in `Startup.Configure`.</span></span>

<span data-ttu-id="32395-324">В примере приложения вызов `RequireHost` в конечной точке в `Startup.Configure` указывает порт управления из конфигурации:</span><span class="sxs-lookup"><span data-stu-id="32395-324">In the sample app, a call to `RequireHost` on the endpoint in `Startup.Configure` specifies the management port from configuration:</span></span>

```csharp
endpoints.MapHealthChecks("/health")
    .RequireHost($"*:{Configuration["ManagementPort"]}");
```

<span data-ttu-id="32395-325">Конечные точки создаются в примере приложения в `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-325">Endpoints are created in the sample app in `Startup.Configure`.</span></span> <span data-ttu-id="32395-326">В следующем примере кода происходит следующее:</span><span class="sxs-lookup"><span data-stu-id="32395-326">In the following example code:</span></span>

* <span data-ttu-id="32395-327">Проверка готовности использует все зарегистрированные проверки с тегом ready.</span><span class="sxs-lookup"><span data-stu-id="32395-327">The readiness check uses all registered checks with the 'ready' tag.</span></span>
* <span data-ttu-id="32395-328">`Predicate` исключает все проверки и возвращает 200 — OK.</span><span class="sxs-lookup"><span data-stu-id="32395-328">The `Predicate` excludes all checks and return a 200-Ok.</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health/ready", new HealthCheckOptions()
    {
        Predicate = (check) => check.Tags.Contains("ready"),
    });

    endpoints.MapHealthChecks("/health/live", new HealthCheckOptions()
    {
        Predicate = (_) => false
    });
}
```

> [!NOTE]
> <span data-ttu-id="32395-329">Можно не создавать файл *launchSettings.json* в примере приложения, явно указав порт управления в коде.</span><span class="sxs-lookup"><span data-stu-id="32395-329">You can avoid creating the *launchSettings.json* file in the sample app by setting the management port explicitly in code.</span></span> <span data-ttu-id="32395-330">В *Program.cs*, где создается <xref:Microsoft.Extensions.Hosting.HostBuilder>, добавьте вызов <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenAnyIP*> и укажите конечную точку порта управления.</span><span class="sxs-lookup"><span data-stu-id="32395-330">In *Program.cs* where the <xref:Microsoft.Extensions.Hosting.HostBuilder> is created, add a call to <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenAnyIP*> and provide the app's management port endpoint.</span></span> <span data-ttu-id="32395-331">В параметре `Configure` файла *ManagementPortStartup.cs* укажите порт управления с помощью `RequireHost`:</span><span class="sxs-lookup"><span data-stu-id="32395-331">In `Configure` of *ManagementPortStartup.cs*, specify the management port with `RequireHost`:</span></span>
>
> <span data-ttu-id="32395-332">*Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="32395-332">*Program.cs*:</span></span>
>
> ```csharp
> return new HostBuilder()
>     .ConfigureWebHostDefaults(webBuilder =>
>     {
>         webBuilder.UseKestrel()
>             .ConfigureKestrel(serverOptions =>
>             {
>                 serverOptions.ListenAnyIP(5001);
>             })
>             .UseStartup(startupType);
>     })
>     .Build();
> ```
>
> <span data-ttu-id="32395-333">*ManagementPortStartup.cs*:</span><span class="sxs-lookup"><span data-stu-id="32395-333">*ManagementPortStartup.cs*:</span></span>
>
> ```csharp
> app.UseEndpoints(endpoints =>
> {
>     endpoints.MapHealthChecks("/health").RequireHost("*:5001");
> });
> ```

<span data-ttu-id="32395-334">Чтобы запустить сценарий с портом управления с помощью примера приложения, выполните следующую команду из папки проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-334">To run the management port configuration scenario using the sample app, execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario port
```

## <a name="distribute-a-health-check-library"></a><span data-ttu-id="32395-335">Распространение библиотеки проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-335">Distribute a health check library</span></span>

<span data-ttu-id="32395-336">Чтобы распространять проверки работоспособности в качестве библиотеки, выполните следующие действия.</span><span class="sxs-lookup"><span data-stu-id="32395-336">To distribute a health check as a library:</span></span>

1. <span data-ttu-id="32395-337">Напишите проверку работоспособности, которая реализует интерфейс <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> как автономный класс.</span><span class="sxs-lookup"><span data-stu-id="32395-337">Write a health check that implements the <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> interface as a standalone class.</span></span> <span data-ttu-id="32395-338">Класс может полагаться на [внедрение зависимостей (DI)](xref:fundamentals/dependency-injection), активацию типов и [именованные параметры](xref:fundamentals/configuration/options) для доступа к данным конфигурации.</span><span class="sxs-lookup"><span data-stu-id="32395-338">The class can rely on [dependency injection (DI)](xref:fundamentals/dependency-injection), type activation, and [named options](xref:fundamentals/configuration/options) to access configuration data.</span></span>

   <span data-ttu-id="32395-339">Логика проверки работоспособности `CheckHealthAsync` предусматривает следующее:</span><span class="sxs-lookup"><span data-stu-id="32395-339">In the health checks logic of `CheckHealthAsync`:</span></span>

   * <span data-ttu-id="32395-340">Использование в методе `data1` и `data2` для выполнения логики проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-340">`data1` and `data2` are used in the method to run the probe's health check logic.</span></span>
   * <span data-ttu-id="32395-341">Обработку `AccessViolationException`.</span><span class="sxs-lookup"><span data-stu-id="32395-341">`AccessViolationException` is handled.</span></span>

   <span data-ttu-id="32395-342">Когда происходит <xref:System.AccessViolationException>, <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.FailureStatus> возвращается вместе с <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult>, что позволяет пользователям настраивать состояние сбоя проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-342">When an <xref:System.AccessViolationException> occurs, the <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.FailureStatus> is returned with the <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> to allow users to configure the health checks failure status.</span></span>

   ```csharp
   using System;
   using System.Threading;
   using System.Threading.Tasks;
   using Microsoft.Extensions.Diagnostics.HealthChecks;

   namespace SampleApp
   {
       public class ExampleHealthCheck : IHealthCheck
       {
           private readonly string _data1;
           private readonly int? _data2;

           public ExampleHealthCheck(string data1, int? data2)
           {
               _data1 = data1 ?? throw new ArgumentNullException(nameof(data1));
               _data2 = data2 ?? throw new ArgumentNullException(nameof(data2));
           }

           public async Task<HealthCheckResult> CheckHealthAsync(
               HealthCheckContext context, CancellationToken cancellationToken)
           {
               try
               {
                   return HealthCheckResult.Healthy();
               }
               catch (AccessViolationException ex)
               {
                   return new HealthCheckResult(
                       context.Registration.FailureStatus,
                       description: "An access violation occurred during the check.",
                       exception: ex,
                       data: null);
               }
           }
       }
   }
   ```

1. <span data-ttu-id="32395-343">Напишите метод расширения с параметрами, который приложение вызывает в своем методе `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-343">Write an extension method with parameters that the consuming app calls in its `Startup.Configure` method.</span></span> <span data-ttu-id="32395-344">В следующем примере предполагается следующая сигнатура метода проверки работоспособности:</span><span class="sxs-lookup"><span data-stu-id="32395-344">In the following example, assume the following health check method signature:</span></span>

   ```csharp
   ExampleHealthCheck(string, string, int )
   ```

   <span data-ttu-id="32395-345">Эта сигнатура указывает, что `ExampleHealthCheck` требуются дополнительные данные для обработки логики проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-345">The preceding signature indicates that the `ExampleHealthCheck` requires additional data to process the health check probe logic.</span></span> <span data-ttu-id="32395-346">Данные передаются делегату, используемому для создания экземпляра проверки работоспособности, когда проверка работоспособности регистрируется с помощью метода расширения.</span><span class="sxs-lookup"><span data-stu-id="32395-346">The data is provided to the delegate used to create the health check instance when the health check is registered with an extension method.</span></span> <span data-ttu-id="32395-347">В следующем примере вызывающая сторона задает:</span><span class="sxs-lookup"><span data-stu-id="32395-347">In the following example, the caller specifies optional:</span></span>

   * <span data-ttu-id="32395-348">Необязательное имя проверки работоспособности (`name`).</span><span class="sxs-lookup"><span data-stu-id="32395-348">health check name (`name`).</span></span> <span data-ttu-id="32395-349">Если `null`, используется `example_health_check`.</span><span class="sxs-lookup"><span data-stu-id="32395-349">If `null`, `example_health_check` is used.</span></span>
   * <span data-ttu-id="32395-350">Строковая точка данных для проверки работоспособности (`data1`).</span><span class="sxs-lookup"><span data-stu-id="32395-350">string data point for the health check (`data1`).</span></span>
   * <span data-ttu-id="32395-351">Целочисленная точка данных для проверки работоспособности (`data2`).</span><span class="sxs-lookup"><span data-stu-id="32395-351">integer data point for the health check (`data2`).</span></span> <span data-ttu-id="32395-352">Если `null`, используется `1`.</span><span class="sxs-lookup"><span data-stu-id="32395-352">If `null`, `1` is used.</span></span>
   * <span data-ttu-id="32395-353">состояние сбоя (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>).</span><span class="sxs-lookup"><span data-stu-id="32395-353">failure status (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>).</span></span> <span data-ttu-id="32395-354">Значение по умолчанию — `null`.</span><span class="sxs-lookup"><span data-stu-id="32395-354">The default is `null`.</span></span> <span data-ttu-id="32395-355">В случае `null` возвращается [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) как состояние сбоя.</span><span class="sxs-lookup"><span data-stu-id="32395-355">If `null`, [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) is reported for a failure status.</span></span>
   * <span data-ttu-id="32395-356">Теги (`IEnumerable<string>`).</span><span class="sxs-lookup"><span data-stu-id="32395-356">tags (`IEnumerable<string>`).</span></span>

   ```csharp
   using System.Collections.Generic;
   using Microsoft.Extensions.Diagnostics.HealthChecks;

   public static class ExampleHealthCheckBuilderExtensions
   {
       const string DefaultName = "example_health_check";

       public static IHealthChecksBuilder AddExampleHealthCheck(
           this IHealthChecksBuilder builder,
           string name = default,
           string data1,
           int data2 = 1,
           HealthStatus? failureStatus = default,
           IEnumerable<string> tags = default)
       {
           return builder.Add(new HealthCheckRegistration(
               name ?? DefaultName,
               sp => new ExampleHealthCheck(data1, data2),
               failureStatus,
               tags));
       }
   }
   ```

## <a name="health-check-publisher"></a><span data-ttu-id="32395-357">Издатель проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-357">Health Check Publisher</span></span>

<span data-ttu-id="32395-358">Когда <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> добавляется в контейнер службы, система проверки работоспособности периодически выполняет ваши проверки работоспособности и вызывает `PublishAsync` с полученным результатом.</span><span class="sxs-lookup"><span data-stu-id="32395-358">When an <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> is added to the service container, the health check system periodically executes your health checks and calls `PublishAsync` with the result.</span></span> <span data-ttu-id="32395-359">Это полезно в ситуации принудительной передачи данных мониторинга работоспособности, когда ожидается, что каждый процесс периодически вызывает систему мониторинга для определения работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-359">This is useful in a push-based health monitoring system scenario that expects each process to call the monitoring system periodically in order to determine health.</span></span>

<span data-ttu-id="32395-360">Интерфейс <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> содержит один метод:</span><span class="sxs-lookup"><span data-stu-id="32395-360">The <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> interface has a single method:</span></span>

```csharp
Task PublishAsync(HealthReport report, CancellationToken cancellationToken);
```

<span data-ttu-id="32395-361"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions> позволяет задать следующие параметры:</span><span class="sxs-lookup"><span data-stu-id="32395-361"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions> allow you to set:</span></span>

* <span data-ttu-id="32395-362"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Delay>. Исходная задержка, которая срабатывает после запуска приложения и перед выполнением экземпляров <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>.</span><span class="sxs-lookup"><span data-stu-id="32395-362"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Delay>: The initial delay applied after the app starts before executing <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> instances.</span></span> <span data-ttu-id="32395-363">Задержка применяется при запуске один раз и не распространяется на все последующие итерации.</span><span class="sxs-lookup"><span data-stu-id="32395-363">The delay is applied once at startup and doesn't apply to subsequent iterations.</span></span> <span data-ttu-id="32395-364">Значение по умолчанию — пять секунд.</span><span class="sxs-lookup"><span data-stu-id="32395-364">The default value is five seconds.</span></span>
* <span data-ttu-id="32395-365"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Period>. Период выполнения <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>.</span><span class="sxs-lookup"><span data-stu-id="32395-365"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Period>: The period of <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> execution.</span></span> <span data-ttu-id="32395-366">Значение по умолчанию - 30 секунды.</span><span class="sxs-lookup"><span data-stu-id="32395-366">The default value is 30 seconds.</span></span>
* <span data-ttu-id="32395-367"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate>. Если <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate> имеет значение `null` (по умолчанию), служба издателя проверки работоспособности выполняет все зарегистрированные проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-367"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate>: If <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate> is `null` (default), the health check publisher service runs all registered health checks.</span></span> <span data-ttu-id="32395-368">Чтобы выполнить ряд проверок работоспособности, укажите функцию, которая отфильтровывает нужный набор.</span><span class="sxs-lookup"><span data-stu-id="32395-368">To run a subset of health checks, provide a function that filters the set of checks.</span></span> <span data-ttu-id="32395-369">Предикат вычисляется в каждом периоде.</span><span class="sxs-lookup"><span data-stu-id="32395-369">The predicate is evaluated each period.</span></span>
* <span data-ttu-id="32395-370"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Timeout>. Время ожидания для выполнения проверок работоспособности для всех экземпляров <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>.</span><span class="sxs-lookup"><span data-stu-id="32395-370"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Timeout>: The timeout for executing the health checks for all <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> instances.</span></span> <span data-ttu-id="32395-371">Используйте <xref:System.Threading.Timeout.InfiniteTimeSpan>, чтобы выполнить проверки без задержки.</span><span class="sxs-lookup"><span data-stu-id="32395-371">Use <xref:System.Threading.Timeout.InfiniteTimeSpan> to execute without a timeout.</span></span> <span data-ttu-id="32395-372">Значение по умолчанию - 30 секунды.</span><span class="sxs-lookup"><span data-stu-id="32395-372">The default value is 30 seconds.</span></span>

<span data-ttu-id="32395-373">В примере приложения `ReadinessPublisher` является реализацией <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>.</span><span class="sxs-lookup"><span data-stu-id="32395-373">In the sample app, `ReadinessPublisher` is an <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> implementation.</span></span> <span data-ttu-id="32395-374">Состояние проверки работоспособности записывается для каждой проверки на таком уровне журнала:</span><span class="sxs-lookup"><span data-stu-id="32395-374">The health check status is logged for each check at a log level of:</span></span>

* <span data-ttu-id="32395-375">на уровне сведений (<xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*>), если состояние проверки работоспособности — <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Healthy>;</span><span class="sxs-lookup"><span data-stu-id="32395-375">Information (<xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*>) if the health checks status is <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Healthy>.</span></span>
* <span data-ttu-id="32395-376">на уровне ошибки (<xref:Microsoft.Extensions.Logging.LoggerExtensions.LogError*>), если состояние соответствует <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Degraded> или <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Unhealthy>.</span><span class="sxs-lookup"><span data-stu-id="32395-376">Error (<xref:Microsoft.Extensions.Logging.LoggerExtensions.LogError*>) if the status is either <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Degraded> or <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Unhealthy>.</span></span>

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/ReadinessPublisher.cs?name=snippet_ReadinessPublisher&highlight=18-27)]

<span data-ttu-id="32395-377">В примере приложения `LivenessProbeStartup` проверка готовности `StartupHostedService` использует задержку запуска в две секунды и выполняет проверку каждые 30 секунд.</span><span class="sxs-lookup"><span data-stu-id="32395-377">In the sample app's `LivenessProbeStartup` example, the `StartupHostedService` readiness check has a two second startup delay and runs the check every 30 seconds.</span></span> <span data-ttu-id="32395-378">Чтобы активировать реализацию <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>, этот пример регистрирует `ReadinessPublisher` как отдельную службу в контейнере [внедрения зависимостей](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="32395-378">To activate the <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> implementation, the sample registers `ReadinessPublisher` as a singleton service in the [dependency injection (DI)](xref:fundamentals/dependency-injection) container:</span></span>

[!code-csharp[](health-checks/samples/3.x/HealthChecksSample/LivenessProbeStartup.cs?name=snippet_ConfigureServices)]

> [!NOTE]
> <span data-ttu-id="32395-379">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) содержит издатели для нескольких систем, в том числе [Application Insights](/azure/application-insights/app-insights-overview).</span><span class="sxs-lookup"><span data-stu-id="32395-379">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) includes publishers for several systems, including [Application Insights](/azure/application-insights/app-insights-overview).</span></span>
>
> <span data-ttu-id="32395-380">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) не поддерживается корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="32395-380">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) isn't maintained or supported by Microsoft.</span></span>

## <a name="restrict-health-checks-with-mapwhen"></a><span data-ttu-id="32395-381">Ограничение проверок работоспособности с помощью MapWhen</span><span class="sxs-lookup"><span data-stu-id="32395-381">Restrict health checks with MapWhen</span></span>

<span data-ttu-id="32395-382">Используйте <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen*> для условного ветвления конвейера запросов для конечных точек проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-382">Use <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen*> to conditionally branch the request pipeline for health check endpoints.</span></span>

<span data-ttu-id="32395-383">В следующем примере `MapWhen` выполняет ветвление конвейера запросов для активации ПО промежуточного слоя для проверки работоспособности, если для конечной точки `api/HealthCheck` поступает запрос GET:</span><span class="sxs-lookup"><span data-stu-id="32395-383">In the following example, `MapWhen` branches the request pipeline to activate Health Checks Middleware if a GET request is received for the `api/HealthCheck` endpoint:</span></span>

```csharp
app.MapWhen(
    context => context.Request.Method == HttpMethod.Get.Method && 
        context.Request.Path.StartsWith("/api/HealthCheck"),
    builder => builder.UseHealthChecks());

app.UseEndpoints(endpoints =>
{
    endpoints.MapRazorPages();
});
```

<span data-ttu-id="32395-384">Для получения дополнительной информации см. <xref:fundamentals/middleware/index#use-run-and-map>.</span><span class="sxs-lookup"><span data-stu-id="32395-384">For more information, see <xref:fundamentals/middleware/index#use-run-and-map>.</span></span>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="32395-385">ASP.NET Core предоставляет ПО промежуточного слоя и библиотеки для создания отчетов о работоспособности компонентов инфраструктуры приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-385">ASP.NET Core offers Health Checks Middleware and libraries for reporting the health of app infrastructure components.</span></span>

<span data-ttu-id="32395-386">Проверки работоспособности предоставляются приложением как конечные точки HTTP.</span><span class="sxs-lookup"><span data-stu-id="32395-386">Health checks are exposed by an app as HTTP endpoints.</span></span> <span data-ttu-id="32395-387">Конечные точки проверки работоспособности можно настроить для различных сценариев в реальном времени мониторинга:</span><span class="sxs-lookup"><span data-stu-id="32395-387">Health check endpoints can be configured for a variety of real-time monitoring scenarios:</span></span>

* <span data-ttu-id="32395-388">Проверки работоспособности можно использовать с оркестраторами контейнеров и подсистемами балансировки нагрузки, чтобы проверять состояние приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-388">Health probes can be used by container orchestrators and load balancers to check an app's status.</span></span> <span data-ttu-id="32395-389">Например, оркестратор контейнеров может реагировать на неуспешную проверку работоспособности, остановив последовательное развертывание или перезапустив контейнер.</span><span class="sxs-lookup"><span data-stu-id="32395-389">For example, a container orchestrator may respond to a failing health check by halting a rolling deployment or restarting a container.</span></span> <span data-ttu-id="32395-390">Балансировщик нагрузки может реагировать на неработоспособное приложение путем перенаправления трафика от неисправного экземпляра к работающему экземпляру.</span><span class="sxs-lookup"><span data-stu-id="32395-390">A load balancer might react to an unhealthy app by routing traffic away from the failing instance to a healthy instance.</span></span>
* <span data-ttu-id="32395-391">Использование памяти, диска и других ресурсов физического сервера можно отслеживать с точки зрения работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-391">Use of memory, disk, and other physical server resources can be monitored for healthy status.</span></span>
* <span data-ttu-id="32395-392">Проверки работоспособности позволяют проверять зависимости приложения, такие как базы данных и конечные точки внешних служб, чтобы убедиться в доступности и нормальной работе.</span><span class="sxs-lookup"><span data-stu-id="32395-392">Health checks can test an app's dependencies, such as databases and external service endpoints, to confirm availability and normal functioning.</span></span>

<span data-ttu-id="32395-393">[Просмотреть или скачать образец кода](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/health-checks/samples) ([как скачивать](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="32395-393">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/health-checks/samples) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="32395-394">Пример приложения включает примеры сценариев, описанных в этом разделе.</span><span class="sxs-lookup"><span data-stu-id="32395-394">The sample app includes examples of the scenarios described in this topic.</span></span> <span data-ttu-id="32395-395">Чтобы запустить пример приложения для заданного сценария, используйте команду [dotnet run](/dotnet/core/tools/dotnet-run) из папки проекта в командной строке.</span><span class="sxs-lookup"><span data-stu-id="32395-395">To run the sample app for a given scenario, use the [dotnet run](/dotnet/core/tools/dotnet-run) command from the project's folder in a command shell.</span></span> <span data-ttu-id="32395-396">См. файл *README.md* приложения и описания сценариев в этом разделе, чтобы получить сведения о том, как использовать пример приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-396">See the sample app's *README.md* file and the scenario descriptions in this topic for details on how to use the sample app.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="32395-397">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="32395-397">Prerequisites</span></span>

<span data-ttu-id="32395-398">Проверки работоспособности обычно используются в оркестраторе контейнеров или внешней службе мониторинга для проверки состояния приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-398">Health checks are usually used with an external monitoring service or container orchestrator to check the status of an app.</span></span> <span data-ttu-id="32395-399">Перед добавлением проверки работоспособности приложения выберите используемую систему мониторинга.</span><span class="sxs-lookup"><span data-stu-id="32395-399">Before adding health checks to an app, decide on which monitoring system to use.</span></span> <span data-ttu-id="32395-400">Система мониторинга определяет, какие виды проверки работоспособности создавать и как настроить их конечные точки.</span><span class="sxs-lookup"><span data-stu-id="32395-400">The monitoring system dictates what types of health checks to create and how to configure their endpoints.</span></span>

<span data-ttu-id="32395-401">Добавьте ссылку на [метапакет Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) или ссылку на пакет [Microsoft.AspNetCore.Diagnostics.HealthCheck](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.HealthChecks).</span><span class="sxs-lookup"><span data-stu-id="32395-401">Reference the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) or add a package reference to the [Microsoft.AspNetCore.Diagnostics.HealthChecks](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.HealthChecks) package.</span></span>

<span data-ttu-id="32395-402">В примере приложения приведен код запуска для демонстрации проверки работоспособности для нескольких сценариев.</span><span class="sxs-lookup"><span data-stu-id="32395-402">The sample app provides startup code to demonstrate health checks for several scenarios.</span></span> <span data-ttu-id="32395-403">Сценарий [проверки базы данных](#database-probe) проверяет работоспособность подключения базы данных с помощью [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks).</span><span class="sxs-lookup"><span data-stu-id="32395-403">The [database probe](#database-probe) scenario checks the health of a database connection using [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks).</span></span> <span data-ttu-id="32395-404">Сценарий [проверки DbContext](#entity-framework-core-dbcontext-probe) проверяет базу данных с помощью EF Core `DbContext`.</span><span class="sxs-lookup"><span data-stu-id="32395-404">The [DbContext probe](#entity-framework-core-dbcontext-probe) scenario checks a database using an EF Core `DbContext`.</span></span> <span data-ttu-id="32395-405">Чтобы изучить сценарии для баз данных, пример приложения выполняет следующие действия.</span><span class="sxs-lookup"><span data-stu-id="32395-405">To explore the database scenarios, the sample app:</span></span>

* <span data-ttu-id="32395-406">Создает базу данных и указывает ее строку подключения в файле приложения *appsettings.json*.</span><span class="sxs-lookup"><span data-stu-id="32395-406">Creates a database and provides its connection string in the *appsettings.json* file.</span></span>
* <span data-ttu-id="32395-407">Содержит следующие ссылки на пакеты в своем файле проекта.</span><span class="sxs-lookup"><span data-stu-id="32395-407">Has the following package references in its project file:</span></span>
  * [<span data-ttu-id="32395-408">AspNetCore.HealthChecks.SqlServer</span><span class="sxs-lookup"><span data-stu-id="32395-408">AspNetCore.HealthChecks.SqlServer</span></span>](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/)
  * [<span data-ttu-id="32395-409">Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore</span><span class="sxs-lookup"><span data-stu-id="32395-409">Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore</span></span>](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/)

> [!NOTE]
> <span data-ttu-id="32395-410">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) не поддерживается корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="32395-410">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) isn't maintained or supported by Microsoft.</span></span>

<span data-ttu-id="32395-411">В другом сценарии проверки работоспособности демонстрируется способ фильтрации проверок работоспособности по порту управления.</span><span class="sxs-lookup"><span data-stu-id="32395-411">Another health check scenario demonstrates how to filter health checks to a management port.</span></span> <span data-ttu-id="32395-412">Пример приложения требует создать файл *Properties/launchSettings.json*, который включает в себя URL-адрес управления и порт управления.</span><span class="sxs-lookup"><span data-stu-id="32395-412">The sample app requires you to create a *Properties/launchSettings.json* file that includes the management URL and management port.</span></span> <span data-ttu-id="32395-413">Дополнительные сведения см. в разделе [Фильтр по портам](#filter-by-port).</span><span class="sxs-lookup"><span data-stu-id="32395-413">For more information, see the [Filter by port](#filter-by-port) section.</span></span>

## <a name="basic-health-probe"></a><span data-ttu-id="32395-414">Базовая проверка работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-414">Basic health probe</span></span>

<span data-ttu-id="32395-415">Для многих приложений достаточно конфигурации базовой проверки работоспособности, которая сообщает о доступности приложения для обработки запросов (*жизнеспособности*).</span><span class="sxs-lookup"><span data-stu-id="32395-415">For many apps, a basic health probe configuration that reports the app's availability to process requests (*liveness*) is sufficient to discover the status of the app.</span></span>

<span data-ttu-id="32395-416">Базовая конфигурация регистрирует службы проверки работоспособности и вызывает ПО промежуточного слоя для получения ответа о работоспособности в конечной точке по URL-адресу.</span><span class="sxs-lookup"><span data-stu-id="32395-416">The basic configuration registers health check services and calls the Health Checks Middleware to respond at a URL endpoint with a health response.</span></span> <span data-ttu-id="32395-417">По умолчанию отдельные проверки работоспособности не регистрируются для тестирования какой-либо конкретной зависимости или подсистемы.</span><span class="sxs-lookup"><span data-stu-id="32395-417">By default, no specific health checks are registered to test any particular dependency or subsystem.</span></span> <span data-ttu-id="32395-418">Приложение считается исправным, если оно способно отвечать на запросы по URL-адресу конечной точки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-418">The app is considered healthy if it's capable of responding at the health endpoint URL.</span></span> <span data-ttu-id="32395-419">Модуль записи ответа по умолчанию передает состояние (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) в виде обычного текста в ответе клиенту. Это может быть состояние [HealthStatus.Healthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus), [HealthStatus.Degraded](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) или [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus).</span><span class="sxs-lookup"><span data-stu-id="32395-419">The default response writer writes the status (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) as a plaintext response back to the client, indicating either a [HealthStatus.Healthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus), [HealthStatus.Degraded](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) or [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) status.</span></span>

<span data-ttu-id="32395-420">Зарегистрируйте службы проверки работоспособности при помощи <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> в `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="32395-420">Register health check services with <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="32395-421">Добавьте конечную точку для ПО промежуточного слоя для проверки работоспособности с помощью <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> в конвейер обработки запросов `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-421">Add an endpoint for Health Checks Middleware with <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> in the request processing pipeline of `Startup.Configure`.</span></span>

<span data-ttu-id="32395-422">В примере приложения конечная точка проверки работоспособности создается в `/health` (*BasicStartup.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-422">In the sample app, the health check endpoint is created at `/health` (*BasicStartup.cs*):</span></span>

```csharp
public class BasicStartup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddHealthChecks();
    }

    public void Configure(IApplicationBuilder app)
    {
        app.UseHealthChecks("/health");
    }
}
```

<span data-ttu-id="32395-423">Чтобы запустить сценарий базовой конфигурации с помощью примера приложения, выполните следующую команду из папки проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-423">To run the basic configuration scenario using the sample app, execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario basic
```

### <a name="docker-example"></a><span data-ttu-id="32395-424">Пример для Docker</span><span class="sxs-lookup"><span data-stu-id="32395-424">Docker example</span></span>

<span data-ttu-id="32395-425">[Docker](xref:host-and-deploy/docker/index) предлагает встроенную директиву `HEALTHCHECK`, которую можно вызвать для проверки состояния приложения, использующего базовую конфигурацию проверки работоспособности:</span><span class="sxs-lookup"><span data-stu-id="32395-425">[Docker](xref:host-and-deploy/docker/index) offers a built-in `HEALTHCHECK` directive that can be used to check the status of an app that uses the basic health check configuration:</span></span>

```
HEALTHCHECK CMD curl --fail http://localhost:5000/health || exit
```

## <a name="create-health-checks"></a><span data-ttu-id="32395-426">Создание проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-426">Create health checks</span></span>

<span data-ttu-id="32395-427">Проверки работоспособности создаются путем реализации интерфейса <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck>.</span><span class="sxs-lookup"><span data-stu-id="32395-427">Health checks are created by implementing the <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> interface.</span></span> <span data-ttu-id="32395-428">Метод <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck.CheckHealthAsync*> возвращает <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult>, указывая состояние работоспособности как `Healthy`, `Degraded` или `Unhealthy`.</span><span class="sxs-lookup"><span data-stu-id="32395-428">The <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck.CheckHealthAsync*> method returns a <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> that indicates the health as `Healthy`, `Degraded`, or `Unhealthy`.</span></span> <span data-ttu-id="32395-429">Результат записывается в ответ в виде обычного текста с кодом состояния, который можно настроить (конфигурация описана в разделе [Варианты проверки работоспособности](#health-check-options)).</span><span class="sxs-lookup"><span data-stu-id="32395-429">The result is written as a plaintext response with a configurable status code (configuration is described in the [Health check options](#health-check-options) section).</span></span> <span data-ttu-id="32395-430"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> также может возвращать необязательные пары "ключ-значение".</span><span class="sxs-lookup"><span data-stu-id="32395-430"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> can also return optional key-value pairs.</span></span>

### <a name="example-health-check"></a><span data-ttu-id="32395-431">Пример проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-431">Example health check</span></span>

<span data-ttu-id="32395-432">Следующий класс `ExampleHealthCheck` демонстрирует структуру процесса проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-432">The following `ExampleHealthCheck` class demonstrates the layout of a health check.</span></span> <span data-ttu-id="32395-433">Логика проверок работоспособности помещается в метод `CheckHealthAsync`.</span><span class="sxs-lookup"><span data-stu-id="32395-433">The health checks logic is placed in the `CheckHealthAsync` method.</span></span> <span data-ttu-id="32395-434">В следующем примере для `true` задается фиктивная переменная `healthCheckResultHealthy`.</span><span class="sxs-lookup"><span data-stu-id="32395-434">The following example sets a dummy variable, `healthCheckResultHealthy`, to `true`.</span></span> <span data-ttu-id="32395-435">Если для `healthCheckResultHealthy` задано значение `false`, возвращается состояние [HealthCheckResult.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult.Unhealthy*).</span><span class="sxs-lookup"><span data-stu-id="32395-435">If the value of `healthCheckResultHealthy` is set to `false`, the [HealthCheckResult.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult.Unhealthy*) status is returned.</span></span>

```csharp
public class ExampleHealthCheck : IHealthCheck
{
    public Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        var healthCheckResultHealthy = true;

        if (healthCheckResultHealthy)
        {
            return Task.FromResult(
                HealthCheckResult.Healthy("The check indicates a healthy result."));
        }

        return Task.FromResult(
            HealthCheckResult.Unhealthy("The check indicates an unhealthy result."));
    }
}
```

### <a name="register-health-check-services"></a><span data-ttu-id="32395-436">Зарегистрируйте службы проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-436">Register health check services</span></span>

<span data-ttu-id="32395-437">Тип `ExampleHealthCheck` добавляется к службам проверки работоспособности в `Startup.ConfigureServices` с помощью <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*>:</span><span class="sxs-lookup"><span data-stu-id="32395-437">The `ExampleHealthCheck` type is added to health check services in `Startup.ConfigureServices` with <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*>:</span></span>

```csharp
services.AddHealthChecks()
    .AddCheck<ExampleHealthCheck>("example_health_check");
```

<span data-ttu-id="32395-438">Перегрузка <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*>, показанная в следующем примере, устанавливает состояние сбоя (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) для отчета в ситуации, когда проверка работоспособности сообщает о сбое.</span><span class="sxs-lookup"><span data-stu-id="32395-438">The <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> overload shown in the following example sets the failure status (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) to report when the health check reports a failure.</span></span> <span data-ttu-id="32395-439">Если состояние сбоя имеет значение `null` (по умолчанию), сообщается [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus).</span><span class="sxs-lookup"><span data-stu-id="32395-439">If the failure status is set to `null` (default), [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) is reported.</span></span> <span data-ttu-id="32395-440">Эта перегрузка — полезный сценарий для авторов библиотек, где состояние сбоя от библиотеки особо учитывается приложением при сбое проверки работоспособности, если реализация проверки работоспособности учитывает этот параметр.</span><span class="sxs-lookup"><span data-stu-id="32395-440">This overload is a useful scenario for library authors, where the failure status indicated by the library is enforced by the app when a health check failure occurs if the health check implementation honors the setting.</span></span>

<span data-ttu-id="32395-441">*Теги* можно использовать для фильтрации проверок работоспособности (описаны далее в разделе [Фильтрация проверок работоспособности](#filter-health-checks)).</span><span class="sxs-lookup"><span data-stu-id="32395-441">*Tags* can be used to filter health checks (described further in the [Filter health checks](#filter-health-checks) section).</span></span>

```csharp
services.AddHealthChecks()
    .AddCheck<ExampleHealthCheck>(
        "example_health_check",
        failureStatus: HealthStatus.Degraded,
        tags: new[] { "example" });
```

<span data-ttu-id="32395-442"><xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> также может выполнять лямбда-функции.</span><span class="sxs-lookup"><span data-stu-id="32395-442"><xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> can also execute a lambda function.</span></span> <span data-ttu-id="32395-443">В следующем примере `Startup.ConfigureServices` имя проверки работоспособности указано как `Example`, а проверка всегда возвращает работоспособное состояние:</span><span class="sxs-lookup"><span data-stu-id="32395-443">In the following `Startup.ConfigureServices` example, the health check name is specified as `Example` and the check always returns a healthy state:</span></span>

```csharp
services.AddHealthChecks()
    .AddCheck("Example", () =>
        HealthCheckResult.Healthy("Example is OK!"), tags: new[] { "example" });
```

### <a name="use-health-checks-middleware"></a><span data-ttu-id="32395-444">Используйте ПО промежуточного слоя для проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-444">Use Health Checks Middleware</span></span>

<span data-ttu-id="32395-445">В `Startup.Configure` вызовите <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> в конвейере обработки с URL-адресом конечной точки или относительным путем:</span><span class="sxs-lookup"><span data-stu-id="32395-445">In `Startup.Configure`, call <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> in the processing pipeline with the endpoint URL or relative path:</span></span>

```csharp
app.UseHealthChecks("/health");
```

<span data-ttu-id="32395-446">Если проверки работоспособности должны прослушивать определенный порт, используйте перегрузку <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*>, чтобы задать порт (описаны далее в разделе [Фильтр по портам](#filter-by-port)):</span><span class="sxs-lookup"><span data-stu-id="32395-446">If the health checks should listen on a specific port, use an overload of <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> to set the port (described further in the [Filter by port](#filter-by-port) section):</span></span>

```csharp
app.UseHealthChecks("/health", port: 8000);
```

## <a name="health-check-options"></a><span data-ttu-id="32395-447">Варианты проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-447">Health check options</span></span>

<span data-ttu-id="32395-448"><xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions> предоставляет возможность настройки поведения для проверки работоспособности:</span><span class="sxs-lookup"><span data-stu-id="32395-448"><xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions> provide an opportunity to customize health check behavior:</span></span>

* [<span data-ttu-id="32395-449">Фильтрация проверок работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-449">Filter health checks</span></span>](#filter-health-checks)
* [<span data-ttu-id="32395-450">Настройка кода состояния HTTP</span><span class="sxs-lookup"><span data-stu-id="32395-450">Customize the HTTP status code</span></span>](#customize-the-http-status-code)
* [<span data-ttu-id="32395-451">Подавление заголовков кэша</span><span class="sxs-lookup"><span data-stu-id="32395-451">Suppress cache headers</span></span>](#suppress-cache-headers)
* [<span data-ttu-id="32395-452">Настройка выходных данных</span><span class="sxs-lookup"><span data-stu-id="32395-452">Customize output</span></span>](#customize-output)

### <a name="filter-health-checks"></a><span data-ttu-id="32395-453">Фильтрация проверок работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-453">Filter health checks</span></span>

<span data-ttu-id="32395-454">По умолчанию ПО промежуточного слоя для проверки работоспособности выполняет все зарегистрированные проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-454">By default, Health Checks Middleware runs all registered health checks.</span></span> <span data-ttu-id="32395-455">Чтобы выполнить подмножество проверок работоспособности, укажите функцию, которая возвращает логическое значение через параметр <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate>.</span><span class="sxs-lookup"><span data-stu-id="32395-455">To run a subset of health checks, provide a function that returns a boolean to the <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate> option.</span></span> <span data-ttu-id="32395-456">В следующем примере проверка работоспособности `Bar` отфильтровывается по тегу (`bar_tag`) в условном операторе функции, где `true` возвращается только в том случае, если свойство проверки работоспособности <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.Tags> соответствует `foo_tag` или `baz_tag`:</span><span class="sxs-lookup"><span data-stu-id="32395-456">In the following example, the `Bar` health check is filtered out by its tag (`bar_tag`) in the function's conditional statement, where `true` is only returned if the health check's <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.Tags> property matches `foo_tag` or `baz_tag`:</span></span>

```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using Microsoft.Extensions.Diagnostics.HealthChecks;

public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddCheck("Foo", () =>
            HealthCheckResult.Healthy("Foo is OK!"), tags: new[] { "foo_tag" })
        .AddCheck("Bar", () =>
            HealthCheckResult.Unhealthy("Bar is unhealthy!"), 
                tags: new[] { "bar_tag" })
        .AddCheck("Baz", () =>
            HealthCheckResult.Healthy("Baz is OK!"), tags: new[] { "baz_tag" });
}

public void Configure(IApplicationBuilder app)
{
    app.UseHealthChecks("/health", new HealthCheckOptions()
    {
        Predicate = (check) => check.Tags.Contains("foo_tag") ||
            check.Tags.Contains("baz_tag")
    });
}
```

### <a name="customize-the-http-status-code"></a><span data-ttu-id="32395-457">Настройка кода состояния HTTP</span><span class="sxs-lookup"><span data-stu-id="32395-457">Customize the HTTP status code</span></span>

<span data-ttu-id="32395-458">Используйте <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResultStatusCodes> для настройки сопоставления состояний работоспособности и кодов состояния HTTP.</span><span class="sxs-lookup"><span data-stu-id="32395-458">Use <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResultStatusCodes> to customize the mapping of health status to HTTP status codes.</span></span> <span data-ttu-id="32395-459">Следующие назначения <xref:Microsoft.AspNetCore.Http.StatusCodes> являются значениями по умолчанию, используемыми ПО промежуточного слоя.</span><span class="sxs-lookup"><span data-stu-id="32395-459">The following <xref:Microsoft.AspNetCore.Http.StatusCodes> assignments are the default values used by the middleware.</span></span> <span data-ttu-id="32395-460">Измените значения кодов состояния в соответствии с требованиями.</span><span class="sxs-lookup"><span data-stu-id="32395-460">Change the status code values to meet your requirements.</span></span>

<span data-ttu-id="32395-461">В `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="32395-461">In `Startup.Configure`:</span></span>

```csharp
//using Microsoft.AspNetCore.Diagnostics.HealthChecks;
//using Microsoft.Extensions.Diagnostics.HealthChecks;

app.UseHealthChecks("/health", new HealthCheckOptions()
{
    ResultStatusCodes =
    {
        [HealthStatus.Healthy] = StatusCodes.Status200OK,
        [HealthStatus.Degraded] = StatusCodes.Status200OK,
        [HealthStatus.Unhealthy] = StatusCodes.Status503ServiceUnavailable
    }
});
```

### <a name="suppress-cache-headers"></a><span data-ttu-id="32395-462">Подавление заголовков кэша</span><span class="sxs-lookup"><span data-stu-id="32395-462">Suppress cache headers</span></span>

<span data-ttu-id="32395-463"><xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.AllowCachingResponses> определяет, добавляет ли ПО промежуточного слоя HTTP-заголовки в ответ проверки, чтобы предотвратить кэширование ответов.</span><span class="sxs-lookup"><span data-stu-id="32395-463"><xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.AllowCachingResponses> controls whether the Health Checks Middleware adds HTTP headers to a probe response to prevent response caching.</span></span> <span data-ttu-id="32395-464">Если значение равно `false` (по умолчанию), ПО промежуточного слоя задает или переопределяет заголовки `Cache-Control`, `Expires` и `Pragma`, чтобы предотвратить кэширование ответов.</span><span class="sxs-lookup"><span data-stu-id="32395-464">If the value is `false` (default), the middleware sets or overrides the `Cache-Control`, `Expires`, and `Pragma` headers to prevent response caching.</span></span> <span data-ttu-id="32395-465">Если значение равно `true`, ПО промежуточного слоя не изменяет заголовки кэша в ответе.</span><span class="sxs-lookup"><span data-stu-id="32395-465">If the value is `true`, the middleware doesn't modify the cache headers of the response.</span></span>

<span data-ttu-id="32395-466">В `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="32395-466">In `Startup.Configure`:</span></span>

```csharp
//using Microsoft.AspNetCore.Diagnostics.HealthChecks;
//using Microsoft.Extensions.Diagnostics.HealthChecks;

app.UseHealthChecks("/health", new HealthCheckOptions()
{
    AllowCachingResponses = false
});
```

### <a name="customize-output"></a><span data-ttu-id="32395-467">Настройка выходных данных</span><span class="sxs-lookup"><span data-stu-id="32395-467">Customize output</span></span>

<span data-ttu-id="32395-468">Параметр <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResponseWriter> возвращает или задает делегат, используемый для записи ответа.</span><span class="sxs-lookup"><span data-stu-id="32395-468">The <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResponseWriter> option gets or sets a delegate used to write the response.</span></span> <span data-ttu-id="32395-469">Делегат по умолчанию записывает минимальный ответ в формате открытого текста со строковым значением [HealthReport.Status](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthReport.Status).</span><span class="sxs-lookup"><span data-stu-id="32395-469">The default delegate writes a minimal plaintext response with the string value of [HealthReport.Status](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthReport.Status).</span></span>

<span data-ttu-id="32395-470">В `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="32395-470">In `Startup.Configure`:</span></span>

```csharp
// using Microsoft.AspNetCore.Diagnostics.HealthChecks;
// using Microsoft.Extensions.Diagnostics.HealthChecks;

app.UseHealthChecks("/health", new HealthCheckOptions()
{
    ResponseWriter = WriteResponse
});
```

<span data-ttu-id="32395-471">Делегат по умолчанию записывает минимальный ответ в формате открытого текста со строковым значением [HealthReport.Status](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthReport.Status).</span><span class="sxs-lookup"><span data-stu-id="32395-471">The default delegate writes a minimal plaintext response with the string value of [HealthReport.Status](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthReport.Status).</span></span> <span data-ttu-id="32395-472">Следующий пользовательский делегат `WriteResponse` выводит настраиваемый ответ JSON:</span><span class="sxs-lookup"><span data-stu-id="32395-472">The following custom delegate, `WriteResponse`, outputs a custom JSON response:</span></span>

```csharp
private static Task WriteResponse(HttpContext httpContext, HealthReport result)
{
    httpContext.Response.ContentType = "application/json";

    var json = new JObject(
        new JProperty("status", result.Status.ToString()),
        new JProperty("results", new JObject(result.Entries.Select(pair =>
            new JProperty(pair.Key, new JObject(
                new JProperty("status", pair.Value.Status.ToString()),
                new JProperty("description", pair.Value.Description),
                new JProperty("data", new JObject(pair.Value.Data.Select(
                    p => new JProperty(p.Key, p.Value))))))))));
    return httpContext.Response.WriteAsync(
        json.ToString(Formatting.Indented));
}
```

<span data-ttu-id="32395-473">Система проверки работоспособности не предоставляет встроенной поддержки сложных форматов возврата JSON, так как формат зависит от выбранной системы мониторинга.</span><span class="sxs-lookup"><span data-stu-id="32395-473">The health checks system doesn't provide built-in support for complex JSON return formats because the format is specific to your choice of monitoring system.</span></span> <span data-ttu-id="32395-474">В приведенном выше примере вы можете настроить `JObject` в соответствии со своими потребностями.</span><span class="sxs-lookup"><span data-stu-id="32395-474">Feel free to customize the `JObject` in the preceding example as necessary to meet your needs.</span></span>

## <a name="database-probe"></a><span data-ttu-id="32395-475">Проверка базы данных</span><span class="sxs-lookup"><span data-stu-id="32395-475">Database probe</span></span>

<span data-ttu-id="32395-476">Проверка работоспособности может задать запрос для выполнения в качестве логического теста для указания того, отвечает ли база данных как обычно.</span><span class="sxs-lookup"><span data-stu-id="32395-476">A health check can specify a database query to run as a boolean test to indicate if the database is responding normally.</span></span>

<span data-ttu-id="32395-477">Пример приложения использует [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks), библиотеку проверки работоспособности для приложений ASP.NET Core, чтобы проверить работоспособность базы данных SQL Server.</span><span class="sxs-lookup"><span data-stu-id="32395-477">The sample app uses [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks), a health check library for ASP.NET Core apps, to perform a health check on a SQL Server database.</span></span> <span data-ttu-id="32395-478">`AspNetCore.Diagnostics.HealthChecks` выполняет запрос `SELECT 1` к базе данных, чтобы подтвердить, что подключение к базе данных находится в работоспособном состоянии.</span><span class="sxs-lookup"><span data-stu-id="32395-478">`AspNetCore.Diagnostics.HealthChecks` executes a `SELECT 1` query against the database to confirm the connection to the database is healthy.</span></span>

> [!WARNING]
> <span data-ttu-id="32395-479">При проверке подключения к базе данных с помощью запроса выберите запрос, возвращающий результат быстро.</span><span class="sxs-lookup"><span data-stu-id="32395-479">When checking a database connection with a query, choose a query that returns quickly.</span></span> <span data-ttu-id="32395-480">Метод запроса связан с риском перегрузки базы данных и снижения ее производительности.</span><span class="sxs-lookup"><span data-stu-id="32395-480">The query approach runs the risk of overloading the database and degrading its performance.</span></span> <span data-ttu-id="32395-481">В большинстве случаев тестовый запрос не является обязательным.</span><span class="sxs-lookup"><span data-stu-id="32395-481">In most cases, running a test query isn't necessary.</span></span> <span data-ttu-id="32395-482">Достаточно успешного подключения к базе данных.</span><span class="sxs-lookup"><span data-stu-id="32395-482">Merely making a successful connection to the database is sufficient.</span></span> <span data-ttu-id="32395-483">Если вам понадобится выполнить запрос, выберите простой запрос SELECT, например `SELECT 1`.</span><span class="sxs-lookup"><span data-stu-id="32395-483">If you find it necessary to run a query, choose a simple SELECT query, such as `SELECT 1`.</span></span>

<span data-ttu-id="32395-484">Добавьте ссылку на пакет [AspNetCore.HealthChecks.SqlServer](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/).</span><span class="sxs-lookup"><span data-stu-id="32395-484">Include a package reference to [AspNetCore.HealthChecks.SqlServer](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/).</span></span>

<span data-ttu-id="32395-485">Укажите допустимую строку подключения к базе данных в файле примера приложения *appsettings.json*.</span><span class="sxs-lookup"><span data-stu-id="32395-485">Supply a valid database connection string in the *appsettings.json* file of the sample app.</span></span> <span data-ttu-id="32395-486">Приложение использует базу данных SQL Server с именем `HealthCheckSample`:</span><span class="sxs-lookup"><span data-stu-id="32395-486">The app uses a SQL Server database named `HealthCheckSample`:</span></span>

[!code-json[](health-checks/samples/2.x/HealthChecksSample/appsettings.json?highlight=3)]

<span data-ttu-id="32395-487">Зарегистрируйте службы проверки работоспособности при помощи <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> в `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="32395-487">Register health check services with <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="32395-488">Пример приложения вызывает метод `AddSqlServer` со строкой подключения базы данных (*DbHealthStartup.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-488">The sample app calls the `AddSqlServer` method with the database's connection string (*DbHealthStartup.cs*):</span></span>

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/DbHealthStartup.cs?name=snippet_ConfigureServices)]

<span data-ttu-id="32395-489">Вызовите ПО промежуточного слоя для проверки работоспособности в конвейере обработки приложения в `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-489">Call Health Checks Middleware in the app processing pipeline in `Startup.Configure`:</span></span>

```csharp
app.UseHealthChecks("/health");
```

<span data-ttu-id="32395-490">Чтобы запустить сценарий проверки базы данных с помощью примера приложения, выполните следующую команду из папки проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-490">To run the database probe scenario using the sample app, execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario db
```

> [!NOTE]
> <span data-ttu-id="32395-491">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) не поддерживается корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="32395-491">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) isn't maintained or supported by Microsoft.</span></span>

## <a name="entity-framework-core-dbcontext-probe"></a><span data-ttu-id="32395-492">Проверка DbContext в Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="32395-492">Entity Framework Core DbContext probe</span></span>

<span data-ttu-id="32395-493">Эта проверка `DbContext` подтверждает, что приложение может взаимодействовать с базой данных, настроенной для EF Core `DbContext`.</span><span class="sxs-lookup"><span data-stu-id="32395-493">The `DbContext` check confirms that the app can communicate with the database configured for an EF Core `DbContext`.</span></span> <span data-ttu-id="32395-494">Проверка `DbContext` поддерживается в приложениях, которые:</span><span class="sxs-lookup"><span data-stu-id="32395-494">The `DbContext` check is supported in apps that:</span></span>

* <span data-ttu-id="32395-495">используют [Entity Framework (EF) Core](/ef/core/);</span><span class="sxs-lookup"><span data-stu-id="32395-495">Use [Entity Framework (EF) Core](/ef/core/).</span></span>
* <span data-ttu-id="32395-496">содержат ссылку на пакет [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/).</span><span class="sxs-lookup"><span data-stu-id="32395-496">Include a package reference to [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/).</span></span>

<span data-ttu-id="32395-497">`AddDbContextCheck<TContext>` регистрирует проверку работоспособности для `DbContext`.</span><span class="sxs-lookup"><span data-stu-id="32395-497">`AddDbContextCheck<TContext>` registers a health check for a `DbContext`.</span></span> <span data-ttu-id="32395-498">`DbContext` передается методу в качестве `TContext`.</span><span class="sxs-lookup"><span data-stu-id="32395-498">The `DbContext` is supplied as the `TContext` to the method.</span></span> <span data-ttu-id="32395-499">Доступна перегрузка, позволяющая настроить состояние ошибки, теги и запрос тестирования.</span><span class="sxs-lookup"><span data-stu-id="32395-499">An overload is available to configure the failure status, tags, and a custom test query.</span></span>

<span data-ttu-id="32395-500">По умолчанию:</span><span class="sxs-lookup"><span data-stu-id="32395-500">By default:</span></span>

* <span data-ttu-id="32395-501">`DbContextHealthCheck` вызывает метод EF Core `CanConnectAsync`.</span><span class="sxs-lookup"><span data-stu-id="32395-501">The `DbContextHealthCheck` calls EF Core's `CanConnectAsync` method.</span></span> <span data-ttu-id="32395-502">Вы можете указать, какая операция выполняется в том случае, когда проверка работоспособности производится с помощью перегрузок метода `AddDbContextCheck`.</span><span class="sxs-lookup"><span data-stu-id="32395-502">You can customize what operation is run when checking health using `AddDbContextCheck` method overloads.</span></span>
* <span data-ttu-id="32395-503">Имя проверки работоспособности — это имя типа `TContext`.</span><span class="sxs-lookup"><span data-stu-id="32395-503">The name of the health check is the name of the `TContext` type.</span></span>

<span data-ttu-id="32395-504">В примере приложения `AppDbContext` предоставляется для `AddDbContextCheck` и регистрируется в качестве службы в `Startup.ConfigureServices` (*DbContextHealthStartup.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-504">In the sample app, `AppDbContext` is provided to `AddDbContextCheck` and registered as a service in `Startup.ConfigureServices` (*DbContextHealthStartup.cs*):</span></span>

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/DbContextHealthStartup.cs?name=snippet_ConfigureServices)]

<span data-ttu-id="32395-505">В примере приложения `UseHealthChecks` добавляет ПО промежуточного слоя для проверки работоспособности в `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-505">In the sample app, `UseHealthChecks` adds the Health Checks Middleware in `Startup.Configure`.</span></span>

```csharp
app.UseHealthChecks("/health");
```

<span data-ttu-id="32395-506">Прежде чем запускать сценарий проверки `DbContext` с помощью примера приложения, убедитесь, что база данных, указанная в строке подключения, не существует в экземпляре SQL Server.</span><span class="sxs-lookup"><span data-stu-id="32395-506">To run the `DbContext` probe scenario using the sample app, confirm that the database specified by the connection string doesn't exist in the SQL Server instance.</span></span> <span data-ttu-id="32395-507">Если база данных существует, удалите ее.</span><span class="sxs-lookup"><span data-stu-id="32395-507">If the database exists, delete it.</span></span>

<span data-ttu-id="32395-508">Выполните следующую команду в папке проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-508">Execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario dbcontext
```

<span data-ttu-id="32395-509">После запуска приложения проверьте состояние работоспособности, выполнив запрос к конечной точке `/health` в браузере.</span><span class="sxs-lookup"><span data-stu-id="32395-509">After the app is running, check the health status by making a request to the `/health` endpoint in a browser.</span></span> <span data-ttu-id="32395-510">Базы данных и `AppDbContext` не существуют, поэтому приложение дает следующий ответ:</span><span class="sxs-lookup"><span data-stu-id="32395-510">The database and `AppDbContext` don't exist, so app provides the following response:</span></span>

```
Unhealthy
```

<span data-ttu-id="32395-511">Заставьте пример приложения создать базу данных.</span><span class="sxs-lookup"><span data-stu-id="32395-511">Trigger the sample app to create the database.</span></span> <span data-ttu-id="32395-512">Сделайте запрос к `/createdatabase`.</span><span class="sxs-lookup"><span data-stu-id="32395-512">Make a request to `/createdatabase`.</span></span> <span data-ttu-id="32395-513">Приложение выдает ответ:</span><span class="sxs-lookup"><span data-stu-id="32395-513">The app responds:</span></span>

```
Creating the database...
Done!
Navigate to /health to see the health status.
```

<span data-ttu-id="32395-514">Сделайте запрос к конечной точке `/health`.</span><span class="sxs-lookup"><span data-stu-id="32395-514">Make a request to the `/health` endpoint.</span></span> <span data-ttu-id="32395-515">База данных и контекст существуют, поэтому приложение отвечает:</span><span class="sxs-lookup"><span data-stu-id="32395-515">The database and context exist, so app responds:</span></span>

```
Healthy
```

<span data-ttu-id="32395-516">Заставьте пример приложения удалить базу данных.</span><span class="sxs-lookup"><span data-stu-id="32395-516">Trigger the sample app to delete the database.</span></span> <span data-ttu-id="32395-517">Сделайте запрос к `/deletedatabase`.</span><span class="sxs-lookup"><span data-stu-id="32395-517">Make a request to `/deletedatabase`.</span></span> <span data-ttu-id="32395-518">Приложение выдает ответ:</span><span class="sxs-lookup"><span data-stu-id="32395-518">The app responds:</span></span>

```
Deleting the database...
Done!
Navigate to /health to see the health status.
```

<span data-ttu-id="32395-519">Сделайте запрос к конечной точке `/health`.</span><span class="sxs-lookup"><span data-stu-id="32395-519">Make a request to the `/health` endpoint.</span></span> <span data-ttu-id="32395-520">Приложение предоставляет ответ о неработоспособности:</span><span class="sxs-lookup"><span data-stu-id="32395-520">The app provides an unhealthy response:</span></span>

```
Unhealthy
```

## <a name="separate-readiness-and-liveness-probes"></a><span data-ttu-id="32395-521">Разделение проверок готовности и активности</span><span class="sxs-lookup"><span data-stu-id="32395-521">Separate readiness and liveness probes</span></span>

<span data-ttu-id="32395-522">В некоторых сценариях размещения используются пары проверок работоспособности, отличающие два состояния приложения:</span><span class="sxs-lookup"><span data-stu-id="32395-522">In some hosting scenarios, a pair of health checks are used that distinguish two app states:</span></span>

* <span data-ttu-id="32395-523">Приложение работает, но еще не готово к получению запросов.</span><span class="sxs-lookup"><span data-stu-id="32395-523">The app is functioning but not yet ready to receive requests.</span></span> <span data-ttu-id="32395-524">Это состояние *готовности* приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-524">This state is the app's *readiness*.</span></span>
* <span data-ttu-id="32395-525">Приложение работает и отвечает на запросы.</span><span class="sxs-lookup"><span data-stu-id="32395-525">The app is functioning and responding to requests.</span></span> <span data-ttu-id="32395-526">Это состояние *жизнеспособности* приложения.</span><span class="sxs-lookup"><span data-stu-id="32395-526">This state is the app's *liveness*.</span></span>

<span data-ttu-id="32395-527">Проверка готовности обычно выполняет ряд проверок, чтобы определить, все ли подсистемы и ресурсы приложения доступны; она более обширна и требует много времени.</span><span class="sxs-lookup"><span data-stu-id="32395-527">The readiness check usually performs a more extensive and time-consuming set of checks to determine if all of the app's subsystems and resources are available.</span></span> <span data-ttu-id="32395-528">Проверка жизнеспособности лишь быстро определяет, доступно ли приложение для обработки запросов.</span><span class="sxs-lookup"><span data-stu-id="32395-528">A liveness check merely performs a quick check to determine if the app is available to process requests.</span></span> <span data-ttu-id="32395-529">По прохождении проверки готовности приложения нет необходимости создавать чрезмерную нагрузку на приложение с дорогостоящим набором проверок готовности&mdash;дальнейшие проверки требуют лишь проверки жизнеспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-529">After the app passes its readiness check, there's no need to burden the app further with the expensive set of readiness checks&mdash;further checks only require checking for liveness.</span></span>

<span data-ttu-id="32395-530">Пример приложения содержит проверку работоспособности, которая сообщает о завершении длительной задачи запуска [размещенной службы](xref:fundamentals/host/hosted-services).</span><span class="sxs-lookup"><span data-stu-id="32395-530">The sample app contains a health check to report the completion of long-running startup task in a [Hosted Service](xref:fundamentals/host/hosted-services).</span></span> <span data-ttu-id="32395-531">`StartupHostedServiceHealthCheck` предоставляет свойство, `StartupTaskCompleted`, которое размещенная служба может задать как `true` после завершения длительной задачи (*StartupHostedServiceHealthCheck.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-531">The `StartupHostedServiceHealthCheck` exposes a property, `StartupTaskCompleted`, that the hosted service can set to `true` when its long-running task is finished (*StartupHostedServiceHealthCheck.cs*):</span></span>

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/StartupHostedServiceHealthCheck.cs?name=snippet1&highlight=7-11)]

<span data-ttu-id="32395-532">Длительная фоновая задача запущена [размещенной службой](xref:fundamentals/host/hosted-services) (*Services/StartupHostedService*).</span><span class="sxs-lookup"><span data-stu-id="32395-532">The long-running background task is started by a [Hosted Service](xref:fundamentals/host/hosted-services) (*Services/StartupHostedService*).</span></span> <span data-ttu-id="32395-533">По завершении задачи `StartupHostedServiceHealthCheck.StartupTaskCompleted` получает значение `true`:</span><span class="sxs-lookup"><span data-stu-id="32395-533">At the conclusion of the task, `StartupHostedServiceHealthCheck.StartupTaskCompleted` is set to `true`:</span></span>

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/Services/StartupHostedService.cs?name=snippet1&highlight=18-20)]

<span data-ttu-id="32395-534">Проверка работоспособности регистрируется через <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> в `Startup.ConfigureServices` вместе с размещенной службой.</span><span class="sxs-lookup"><span data-stu-id="32395-534">The health check is registered with <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> in `Startup.ConfigureServices` along with the hosted service.</span></span> <span data-ttu-id="32395-535">Поскольку размещенной службе необходимо задать свойство проверки работоспособности, проверка работоспособности также регистрируется в контейнере службы (*LivenessProbeStartup.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-535">Because the hosted service must set the property on the health check, the health check is also registered in the service container (*LivenessProbeStartup.cs*):</span></span>

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/LivenessProbeStartup.cs?name=snippet_ConfigureServices)]

<span data-ttu-id="32395-536">Вызовите ПО промежуточного слоя для проверки работоспособности в конвейере обработки приложения в `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-536">Call Health Checks Middleware in the app processing pipeline in `Startup.Configure`.</span></span> <span data-ttu-id="32395-537">В примере приложения создаются конечные точки проверки работоспособности в `/health/ready` для проверки готовности и в `/health/live` для проверки жизнеспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-537">In the sample app, the health check endpoints are created at `/health/ready` for the readiness check and `/health/live` for the liveness check.</span></span> <span data-ttu-id="32395-538">Проверка готовности фильтрует проверки работоспособности по тегу `ready`.</span><span class="sxs-lookup"><span data-stu-id="32395-538">The readiness check filters health checks to the health check with the `ready` tag.</span></span> <span data-ttu-id="32395-539">Проверка жизнеспособности отфильтровывает `StartupHostedServiceHealthCheck`, возвращая `false` в [HealthCheckOptions.Predicate](xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate) (дополнительные сведения см. в разделе [Фильтрация проверок работоспособности](#filter-health-checks)):</span><span class="sxs-lookup"><span data-stu-id="32395-539">The liveness check filters out the `StartupHostedServiceHealthCheck` by returning `false` in the [HealthCheckOptions.Predicate](xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate) (for more information, see [Filter health checks](#filter-health-checks)):</span></span>

```csharp
app.UseHealthChecks("/health/ready", new HealthCheckOptions()
{
    Predicate = (check) => check.Tags.Contains("ready"), 
});

app.UseHealthChecks("/health/live", new HealthCheckOptions()
{
    Predicate = (_) => false
});
```

<span data-ttu-id="32395-540">Чтобы запустить сценарий конфигурации проверок готовности и жизнеспособности с помощью примера приложения, выполните следующую команду из папки проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-540">To run the readiness/liveness configuration scenario using the sample app, execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario liveness
```

<span data-ttu-id="32395-541">В браузере посетите `/health/ready` несколько раз, пока не пройдет 15 секунд.</span><span class="sxs-lookup"><span data-stu-id="32395-541">In a browser, visit `/health/ready` several times until 15 seconds have passed.</span></span> <span data-ttu-id="32395-542">Проверка работоспособности будет передавать состояние *Unhealthy* первые 15 секунд.</span><span class="sxs-lookup"><span data-stu-id="32395-542">The health check reports *Unhealthy* for the first 15 seconds.</span></span> <span data-ttu-id="32395-543">По истечении 15 секунд конечная точка передает состояние *Healthy*, что означает завершение длительной задачи размещенной службой.</span><span class="sxs-lookup"><span data-stu-id="32395-543">After 15 seconds, the endpoint reports *Healthy*, which reflects the completion of the long-running task by the hosted service.</span></span>

<span data-ttu-id="32395-544">В этом примере также создается издатель проверки работоспособности (реализация <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>), который выполнит первую проверку готовности с задержкой в две секунды.</span><span class="sxs-lookup"><span data-stu-id="32395-544">This example also creates a Health Check Publisher (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> implementation) that runs the first readiness check with a two second delay.</span></span> <span data-ttu-id="32395-545">Дополнительные сведения см. в разделе [об издателе проверки работоспособности](#health-check-publisher).</span><span class="sxs-lookup"><span data-stu-id="32395-545">For more information, see the [Health Check Publisher](#health-check-publisher) section.</span></span>

### <a name="kubernetes-example"></a><span data-ttu-id="32395-546">Пример для Kubernetes</span><span class="sxs-lookup"><span data-stu-id="32395-546">Kubernetes example</span></span>

<span data-ttu-id="32395-547">Использование отдельных проверок готовности и жизнеспособности полезно в таких средах, как [Kubernetes](https://kubernetes.io/).</span><span class="sxs-lookup"><span data-stu-id="32395-547">Using separate readiness and liveness checks is useful in an environment such as [Kubernetes](https://kubernetes.io/).</span></span> <span data-ttu-id="32395-548">В Kubernetes приложениям может потребоваться долго выполнять задачи во время запуска, прежде чем начать принимать запросы; например, проверить доступность основной базы данных.</span><span class="sxs-lookup"><span data-stu-id="32395-548">In Kubernetes, an app might be required to perform time-consuming startup work before accepting requests, such as a test of the underlying database availability.</span></span> <span data-ttu-id="32395-549">Использование отдельных проверок позволяет оркестратору различать, когда приложение работает, но еще не готово, а когда приложение не удалось запустить.</span><span class="sxs-lookup"><span data-stu-id="32395-549">Using separate checks allows the orchestrator to distinguish whether the app is functioning but not yet ready or if the app has failed to start.</span></span> <span data-ttu-id="32395-550">Дополнительные сведения о проверках готовности и жизнеспособности в Kubernetes см. в разделе [Настройка проверок готовности и жизнеспособности](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) в документации по Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="32395-550">For more information on readiness and liveness probes in Kubernetes, see [Configure Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) in the Kubernetes documentation.</span></span>

<span data-ttu-id="32395-551">В следующем примере показана конфигурация проверки готовности для Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="32395-551">The following example demonstrates a Kubernetes readiness probe configuration:</span></span>

```
spec:
  template:
  spec:
    readinessProbe:
      # an http probe
      httpGet:
        path: /health/ready
        port: 80
      # length of time to wait for a pod to initialize
      # after pod startup, before applying health checking
      initialDelaySeconds: 30
      timeoutSeconds: 1
    ports:
      - containerPort: 80
```

## <a name="metric-based-probe-with-a-custom-response-writer"></a><span data-ttu-id="32395-552">Проба на основе метрики с настраиваемым модулем записи ответов</span><span class="sxs-lookup"><span data-stu-id="32395-552">Metric-based probe with a custom response writer</span></span>

<span data-ttu-id="32395-553">Этот пример демонстрирует проверку работоспособности памяти с настраиваемым модулем записи ответов.</span><span class="sxs-lookup"><span data-stu-id="32395-553">The sample app demonstrates a memory health check with a custom response writer.</span></span>

<span data-ttu-id="32395-554">Если приложение использует больше заданного порогового значения памяти (1 ГБ в примере приложения), `MemoryHealthCheck` сообщает о неработоспособном состоянии.</span><span class="sxs-lookup"><span data-stu-id="32395-554">`MemoryHealthCheck` reports an unhealthy status if the app uses more than a given threshold of memory (1 GB in the sample app).</span></span> <span data-ttu-id="32395-555"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> включает сведения от сборщика мусора для приложения (*MemoryHealthCheck.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-555">The <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> includes Garbage Collector (GC) information for the app (*MemoryHealthCheck.cs*):</span></span>

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/MemoryHealthCheck.cs?name=snippet1)]

<span data-ttu-id="32395-556">Зарегистрируйте службы проверки работоспособности при помощи <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> в `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="32395-556">Register health check services with <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="32395-557">Вместо того чтобы включать проверку работоспособности, передав ее в <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*>, `MemoryHealthCheck` регистрируется в качестве службы.</span><span class="sxs-lookup"><span data-stu-id="32395-557">Instead of enabling the health check by passing it to <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*>, the `MemoryHealthCheck` is registered as a service.</span></span> <span data-ttu-id="32395-558">Все зарегистрированные службы <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> доступны в службах проверки работоспособности и ПО промежуточного слоя.</span><span class="sxs-lookup"><span data-stu-id="32395-558">All <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> registered services are available to the health check services and middleware.</span></span> <span data-ttu-id="32395-559">Мы рекомендуем регистрировать службы проверки работоспособности в качестве одноэлементных служб.</span><span class="sxs-lookup"><span data-stu-id="32395-559">We recommend registering health check services as Singleton services.</span></span>

<span data-ttu-id="32395-560">В примере приложения (*CustomWriterStartup.CS*) выполняется следующее:</span><span class="sxs-lookup"><span data-stu-id="32395-560">In the sample app (*CustomWriterStartup.cs*):</span></span>

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_ConfigureServices&highlight=4)]

<span data-ttu-id="32395-561">Вызовите ПО промежуточного слоя для проверки работоспособности в конвейере обработки приложения в `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-561">Call Health Checks Middleware in the app processing pipeline in `Startup.Configure`.</span></span> <span data-ttu-id="32395-562">Делегат `WriteResponse` передается в свойство `ResponseWriter` для вывода настраиваемого ответа JSON при выполнении проверки работоспособности:</span><span class="sxs-lookup"><span data-stu-id="32395-562">A `WriteResponse` delegate is provided to the `ResponseWriter` property to output a custom JSON response when the health check executes:</span></span>

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseHealthChecks("/health", new HealthCheckOptions()
    {
        // This custom writer formats the detailed status as JSON.
        ResponseWriter = WriteResponse
    });
}
```

<span data-ttu-id="32395-563">Метод `WriteResponse` форматирует `CompositeHealthCheckResult` в объект JSON и выдает выходные данные JSON в качестве ответа проверки работоспособности:</span><span class="sxs-lookup"><span data-stu-id="32395-563">The `WriteResponse` method formats the `CompositeHealthCheckResult` into a JSON object and yields JSON output for the health check response:</span></span>

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_WriteResponse)]

<span data-ttu-id="32395-564">Чтобы запустить проверку метрики с настраиваемым модулем записи ответа с помощью примера приложения, выполните следующую команду из папки проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-564">To run the metric-based probe with custom response writer output using the sample app, execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario writer
```

> [!NOTE]
> <span data-ttu-id="32395-565">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) содержит сценарии для проверки работоспособности на основе метрик, включая проверки жизнеспособности для дискового хранилища и максимальных значений.</span><span class="sxs-lookup"><span data-stu-id="32395-565">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) includes metric-based health check scenarios, including disk storage and maximum value liveness checks.</span></span>
>
> <span data-ttu-id="32395-566">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) не поддерживается корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="32395-566">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) isn't maintained or supported by Microsoft.</span></span>

## <a name="filter-by-port"></a><span data-ttu-id="32395-567">Фильтр по портам</span><span class="sxs-lookup"><span data-stu-id="32395-567">Filter by port</span></span>

<span data-ttu-id="32395-568">Вызов <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> с портом ограничивает запросы на проверку работоспособности указанным портом.</span><span class="sxs-lookup"><span data-stu-id="32395-568">Calling <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> with a port restricts health check requests to the port specified.</span></span> <span data-ttu-id="32395-569">Обычно это используется в контейнерной среде с целью предоставления порта для мониторинга служб.</span><span class="sxs-lookup"><span data-stu-id="32395-569">This is typically used in a container environment to expose a port for monitoring services.</span></span>

<span data-ttu-id="32395-570">Пример приложения настраивает порт с помощью [поставщика конфигурации переменных среды](xref:fundamentals/configuration/index#environment-variables-configuration-provider).</span><span class="sxs-lookup"><span data-stu-id="32395-570">The sample app configures the port using the [Environment Variable Configuration Provider](xref:fundamentals/configuration/index#environment-variables-configuration-provider).</span></span> <span data-ttu-id="32395-571">Порт устанавливается в файле *launchSettings.json* и передается поставщику конфигурации с помощью переменной среды.</span><span class="sxs-lookup"><span data-stu-id="32395-571">The port is set in the *launchSettings.json* file and passed to the configuration provider via an environment variable.</span></span> <span data-ttu-id="32395-572">Необходимо также настроить на сервере прослушивание запросов через порт управления.</span><span class="sxs-lookup"><span data-stu-id="32395-572">You must also configure the server to listen to requests on the management port.</span></span>

<span data-ttu-id="32395-573">Чтобы использовать пример приложения для демонстрации настройки портов управления, создайте файл *launchSettings.json* в папке *Properties*.</span><span class="sxs-lookup"><span data-stu-id="32395-573">To use the sample app to demonstrate management port configuration, create the *launchSettings.json* file in a *Properties* folder.</span></span>

<span data-ttu-id="32395-574">Следующий файл примера приложения *Properties/launchSettings.json* отсутствует в файлах проекта примера приложения и должен быть создан вручную:</span><span class="sxs-lookup"><span data-stu-id="32395-574">The following *Properties/launchSettings.json* file in the sample app isn't included in the sample app's project files and must be created manually:</span></span>

```json
{
  "profiles": {
    "SampleApp": {
      "commandName": "Project",
      "commandLineArgs": "",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "ASPNETCORE_URLS": "http://localhost:5000/;http://localhost:5001/",
        "ASPNETCORE_MANAGEMENTPORT": "5001"
      },
      "applicationUrl": "http://localhost:5000/"
    }
  }
}
```

<span data-ttu-id="32395-575">Зарегистрируйте службы проверки работоспособности при помощи <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> в `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="32395-575">Register health check services with <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> in `Startup.ConfigureServices`.</span></span> <span data-ttu-id="32395-576">Вызов <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> задает порт управления (*ManagementPortStartup.cs*):</span><span class="sxs-lookup"><span data-stu-id="32395-576">The call to <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> specifies the management port (*ManagementPortStartup.cs*):</span></span>

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/ManagementPortStartup.cs?name=snippet1&highlight=17)]

> [!NOTE]
> <span data-ttu-id="32395-577">Можно избежать создания файла *launchSettings.json* в примере приложения, явно указав URL-адрес и порт управления в коде.</span><span class="sxs-lookup"><span data-stu-id="32395-577">You can avoid creating the *launchSettings.json* file in the sample app by setting the URLs and management port explicitly in code.</span></span> <span data-ttu-id="32395-578">В *Program.cs*, где создается <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder>, добавьте вызов <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*> и укажите обычную конечную точку ответа приложения и порт конечной точки управления.</span><span class="sxs-lookup"><span data-stu-id="32395-578">In *Program.cs* where the <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder> is created, add a call to <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*> and provide the app's normal response endpoint and the management port endpoint.</span></span> <span data-ttu-id="32395-579">В *ManagementPortStartup.cs*, где вызывается <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*>, явно укажите порт управления.</span><span class="sxs-lookup"><span data-stu-id="32395-579">In *ManagementPortStartup.cs* where <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> is called, specify the management port explicitly.</span></span>
>
> <span data-ttu-id="32395-580">*Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="32395-580">*Program.cs*:</span></span>
>
> ```csharp
> return new WebHostBuilder()
>     .UseConfiguration(config)
>     .UseUrls("http://localhost:5000/;http://localhost:5001/")
>     .ConfigureLogging(builder =>
>     {
>         builder.SetMinimumLevel(LogLevel.Trace);
>         builder.AddConfiguration(config);
>         builder.AddConsole();
>     })
>     .UseKestrel()
>     .UseStartup(startupType)
>     .Build();
> ```
>
> <span data-ttu-id="32395-581">*ManagementPortStartup.cs*:</span><span class="sxs-lookup"><span data-stu-id="32395-581">*ManagementPortStartup.cs*:</span></span>
>
> ```csharp
> app.UseHealthChecks("/health", port: 5001);
> ```

<span data-ttu-id="32395-582">Чтобы запустить сценарий с портом управления с помощью примера приложения, выполните следующую команду из папки проекта в командной строке:</span><span class="sxs-lookup"><span data-stu-id="32395-582">To run the management port configuration scenario using the sample app, execute the following command from the project's folder in a command shell:</span></span>

```dotnetcli
dotnet run --scenario port
```

## <a name="distribute-a-health-check-library"></a><span data-ttu-id="32395-583">Распространение библиотеки проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-583">Distribute a health check library</span></span>

<span data-ttu-id="32395-584">Чтобы распространять проверки работоспособности в качестве библиотеки, выполните следующие действия.</span><span class="sxs-lookup"><span data-stu-id="32395-584">To distribute a health check as a library:</span></span>

1. <span data-ttu-id="32395-585">Напишите проверку работоспособности, которая реализует интерфейс <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> как автономный класс.</span><span class="sxs-lookup"><span data-stu-id="32395-585">Write a health check that implements the <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> interface as a standalone class.</span></span> <span data-ttu-id="32395-586">Класс может полагаться на [внедрение зависимостей (DI)](xref:fundamentals/dependency-injection), активацию типов и [именованные параметры](xref:fundamentals/configuration/options) для доступа к данным конфигурации.</span><span class="sxs-lookup"><span data-stu-id="32395-586">The class can rely on [dependency injection (DI)](xref:fundamentals/dependency-injection), type activation, and [named options](xref:fundamentals/configuration/options) to access configuration data.</span></span>

   <span data-ttu-id="32395-587">Логика проверки работоспособности `CheckHealthAsync` предусматривает следующее:</span><span class="sxs-lookup"><span data-stu-id="32395-587">In the health checks logic of `CheckHealthAsync`:</span></span>

   * <span data-ttu-id="32395-588">Использование в методе `data1` и `data2` для выполнения логики проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-588">`data1` and `data2` are used in the method to run the probe's health check logic.</span></span>
   * <span data-ttu-id="32395-589">Обработку `AccessViolationException`.</span><span class="sxs-lookup"><span data-stu-id="32395-589">`AccessViolationException` is handled.</span></span>

   <span data-ttu-id="32395-590">Когда происходит <xref:System.AccessViolationException>, <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.FailureStatus> возвращается вместе с <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult>, что позволяет пользователям настраивать состояние сбоя проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-590">When an <xref:System.AccessViolationException> occurs, the <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.FailureStatus> is returned with the <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> to allow users to configure the health checks failure status.</span></span>

   ```csharp
   using System;
   using System.Threading;
   using System.Threading.Tasks;
   using Microsoft.Extensions.Diagnostics.HealthChecks;

   public class ExampleHealthCheck : IHealthCheck
   {
       private readonly string _data1;
       private readonly int? _data2;

       public ExampleHealthCheck(string data1, int? data2)
       {
           _data1 = data1 ?? throw new ArgumentNullException(nameof(data1));
           _data2 = data2 ?? throw new ArgumentNullException(nameof(data2));
       }

       public async Task<HealthCheckResult> CheckHealthAsync(
           HealthCheckContext context, CancellationToken cancellationToken)
       {
           try
           {
               return HealthCheckResult.Healthy();
           }
           catch (AccessViolationException ex)
           {
               return new HealthCheckResult(
                   context.Registration.FailureStatus,
                   description: "An access violation occurred during the check.",
                   exception: ex,
                   data: null);
           }
       }
   }
   ```

1. <span data-ttu-id="32395-591">Напишите метод расширения с параметрами, который приложение вызывает в своем методе `Startup.Configure`.</span><span class="sxs-lookup"><span data-stu-id="32395-591">Write an extension method with parameters that the consuming app calls in its `Startup.Configure` method.</span></span> <span data-ttu-id="32395-592">В следующем примере предполагается следующая сигнатура метода проверки работоспособности:</span><span class="sxs-lookup"><span data-stu-id="32395-592">In the following example, assume the following health check method signature:</span></span>

   ```csharp
   ExampleHealthCheck(string, string, int )
   ```

   <span data-ttu-id="32395-593">Эта сигнатура указывает, что `ExampleHealthCheck` требуются дополнительные данные для обработки логики проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-593">The preceding signature indicates that the `ExampleHealthCheck` requires additional data to process the health check probe logic.</span></span> <span data-ttu-id="32395-594">Данные передаются делегату, используемому для создания экземпляра проверки работоспособности, когда проверка работоспособности регистрируется с помощью метода расширения.</span><span class="sxs-lookup"><span data-stu-id="32395-594">The data is provided to the delegate used to create the health check instance when the health check is registered with an extension method.</span></span> <span data-ttu-id="32395-595">В следующем примере вызывающая сторона задает:</span><span class="sxs-lookup"><span data-stu-id="32395-595">In the following example, the caller specifies optional:</span></span>

   * <span data-ttu-id="32395-596">Необязательное имя проверки работоспособности (`name`).</span><span class="sxs-lookup"><span data-stu-id="32395-596">health check name (`name`).</span></span> <span data-ttu-id="32395-597">Если `null`, используется `example_health_check`.</span><span class="sxs-lookup"><span data-stu-id="32395-597">If `null`, `example_health_check` is used.</span></span>
   * <span data-ttu-id="32395-598">Строковая точка данных для проверки работоспособности (`data1`).</span><span class="sxs-lookup"><span data-stu-id="32395-598">string data point for the health check (`data1`).</span></span>
   * <span data-ttu-id="32395-599">Целочисленная точка данных для проверки работоспособности (`data2`).</span><span class="sxs-lookup"><span data-stu-id="32395-599">integer data point for the health check (`data2`).</span></span> <span data-ttu-id="32395-600">Если `null`, используется `1`.</span><span class="sxs-lookup"><span data-stu-id="32395-600">If `null`, `1` is used.</span></span>
   * <span data-ttu-id="32395-601">состояние сбоя (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>).</span><span class="sxs-lookup"><span data-stu-id="32395-601">failure status (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>).</span></span> <span data-ttu-id="32395-602">Значение по умолчанию — `null`.</span><span class="sxs-lookup"><span data-stu-id="32395-602">The default is `null`.</span></span> <span data-ttu-id="32395-603">В случае `null` возвращается [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) как состояние сбоя.</span><span class="sxs-lookup"><span data-stu-id="32395-603">If `null`, [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) is reported for a failure status.</span></span>
   * <span data-ttu-id="32395-604">Теги (`IEnumerable<string>`).</span><span class="sxs-lookup"><span data-stu-id="32395-604">tags (`IEnumerable<string>`).</span></span>

   ```csharp
   using System.Collections.Generic;
   using Microsoft.Extensions.Diagnostics.HealthChecks;

   public static class ExampleHealthCheckBuilderExtensions
   {
       const string DefaultName = "example_health_check";

       public static IHealthChecksBuilder AddExampleHealthCheck(
           this IHealthChecksBuilder builder,
           string name = default,
           string data1,
           int data2 = 1,
           HealthStatus? failureStatus = default,
           IEnumerable<string> tags = default)
       {
           return builder.Add(new HealthCheckRegistration(
               name ?? DefaultName,
               sp => new ExampleHealthCheck(data1, data2),
               failureStatus,
               tags));
       }
   }
   ```

## <a name="health-check-publisher"></a><span data-ttu-id="32395-605">Издатель проверки работоспособности</span><span class="sxs-lookup"><span data-stu-id="32395-605">Health Check Publisher</span></span>

<span data-ttu-id="32395-606">Когда <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> добавляется в контейнер службы, система проверки работоспособности периодически выполняет ваши проверки работоспособности и вызывает `PublishAsync` с полученным результатом.</span><span class="sxs-lookup"><span data-stu-id="32395-606">When an <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> is added to the service container, the health check system periodically executes your health checks and calls `PublishAsync` with the result.</span></span> <span data-ttu-id="32395-607">Это полезно в ситуации принудительной передачи данных мониторинга работоспособности, когда ожидается, что каждый процесс периодически вызывает систему мониторинга для определения работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-607">This is useful in a push-based health monitoring system scenario that expects each process to call the monitoring system periodically in order to determine health.</span></span>

<span data-ttu-id="32395-608">Интерфейс <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> содержит один метод:</span><span class="sxs-lookup"><span data-stu-id="32395-608">The <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> interface has a single method:</span></span>

```csharp
Task PublishAsync(HealthReport report, CancellationToken cancellationToken);
```

<span data-ttu-id="32395-609"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions> позволяет задать следующие параметры:</span><span class="sxs-lookup"><span data-stu-id="32395-609"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions> allow you to set:</span></span>

* <span data-ttu-id="32395-610"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Delay>. Исходная задержка, которая срабатывает после запуска приложения и перед выполнением экземпляров <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>.</span><span class="sxs-lookup"><span data-stu-id="32395-610"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Delay>: The initial delay applied after the app starts before executing <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> instances.</span></span> <span data-ttu-id="32395-611">Задержка применяется при запуске один раз и не распространяется на все последующие итерации.</span><span class="sxs-lookup"><span data-stu-id="32395-611">The delay is applied once at startup and doesn't apply to subsequent iterations.</span></span> <span data-ttu-id="32395-612">Значение по умолчанию — пять секунд.</span><span class="sxs-lookup"><span data-stu-id="32395-612">The default value is five seconds.</span></span>
* <span data-ttu-id="32395-613"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Period>. Период выполнения <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>.</span><span class="sxs-lookup"><span data-stu-id="32395-613"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Period>: The period of <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> execution.</span></span> <span data-ttu-id="32395-614">Значение по умолчанию - 30 секунды.</span><span class="sxs-lookup"><span data-stu-id="32395-614">The default value is 30 seconds.</span></span>
* <span data-ttu-id="32395-615"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate>. Если <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate> имеет значение `null` (по умолчанию), служба издателя проверки работоспособности выполняет все зарегистрированные проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-615"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate>: If <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate> is `null` (default), the health check publisher service runs all registered health checks.</span></span> <span data-ttu-id="32395-616">Чтобы выполнить ряд проверок работоспособности, укажите функцию, которая отфильтровывает нужный набор.</span><span class="sxs-lookup"><span data-stu-id="32395-616">To run a subset of health checks, provide a function that filters the set of checks.</span></span> <span data-ttu-id="32395-617">Предикат вычисляется в каждом периоде.</span><span class="sxs-lookup"><span data-stu-id="32395-617">The predicate is evaluated each period.</span></span>
* <span data-ttu-id="32395-618"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Timeout>. Время ожидания для выполнения проверок работоспособности для всех экземпляров <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>.</span><span class="sxs-lookup"><span data-stu-id="32395-618"><xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Timeout>: The timeout for executing the health checks for all <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> instances.</span></span> <span data-ttu-id="32395-619">Используйте <xref:System.Threading.Timeout.InfiniteTimeSpan>, чтобы выполнить проверки без задержки.</span><span class="sxs-lookup"><span data-stu-id="32395-619">Use <xref:System.Threading.Timeout.InfiniteTimeSpan> to execute without a timeout.</span></span> <span data-ttu-id="32395-620">Значение по умолчанию - 30 секунды.</span><span class="sxs-lookup"><span data-stu-id="32395-620">The default value is 30 seconds.</span></span>

> [!WARNING]
> <span data-ttu-id="32395-621">В выпуске ASP.NET Core 2.2 значение <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Period> не поддерживается в реализации <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>. Здесь задается значение <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Delay>.</span><span class="sxs-lookup"><span data-stu-id="32395-621">In the ASP.NET Core 2.2 release, setting <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Period> isn't honored by the <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> implementation; it sets the value of <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Delay>.</span></span> <span data-ttu-id="32395-622">Эта проблема была устранена в ASP.NET Core 3.0.</span><span class="sxs-lookup"><span data-stu-id="32395-622">This issue has been addressed in ASP.NET Core 3.0.</span></span>

<span data-ttu-id="32395-623">В примере приложения `ReadinessPublisher` является реализацией <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>.</span><span class="sxs-lookup"><span data-stu-id="32395-623">In the sample app, `ReadinessPublisher` is an <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> implementation.</span></span> <span data-ttu-id="32395-624">Состояние проверки работоспособности записывается для каждой проверки на одном из таких уровней журнала:</span><span class="sxs-lookup"><span data-stu-id="32395-624">The health check status is logged for each check as either:</span></span>

* <span data-ttu-id="32395-625">на уровне сведений (<xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*>), если состояние проверки работоспособности — <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Healthy>;</span><span class="sxs-lookup"><span data-stu-id="32395-625">Information (<xref:Microsoft.Extensions.Logging.LoggerExtensions.LogInformation*>) if the health checks status is <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Healthy>.</span></span>
* <span data-ttu-id="32395-626">на уровне ошибки (<xref:Microsoft.Extensions.Logging.LoggerExtensions.LogError*>), если состояние соответствует <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Degraded> или <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Unhealthy>.</span><span class="sxs-lookup"><span data-stu-id="32395-626">Error (<xref:Microsoft.Extensions.Logging.LoggerExtensions.LogError*>) if the status is either <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Degraded> or <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus.Unhealthy>.</span></span>

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/ReadinessPublisher.cs?name=snippet_ReadinessPublisher&highlight=18-27)]

<span data-ttu-id="32395-627">В примере приложения `LivenessProbeStartup` проверка готовности `StartupHostedService` использует задержку запуска в две секунды и выполняет проверку каждые 30 секунд.</span><span class="sxs-lookup"><span data-stu-id="32395-627">In the sample app's `LivenessProbeStartup` example, the `StartupHostedService` readiness check has a two second startup delay and runs the check every 30 seconds.</span></span> <span data-ttu-id="32395-628">Чтобы активировать реализацию <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>, этот пример регистрирует `ReadinessPublisher` как отдельную службу в контейнере [внедрения зависимостей](xref:fundamentals/dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="32395-628">To activate the <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> implementation, the sample registers `ReadinessPublisher` as a singleton service in the [dependency injection (DI)](xref:fundamentals/dependency-injection) container:</span></span>

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/LivenessProbeStartup.cs?name=snippet_ConfigureServices&highlight=12-17,28)]

> [!NOTE]
> <span data-ttu-id="32395-629">Следующий вариант позволяет добавлять экземпляр <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> в контейнер службы, когда один или несколько других размещенных служб уже добавлены в приложение.</span><span class="sxs-lookup"><span data-stu-id="32395-629">The following workaround permits adding an <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> instance to the service container when one or more other hosted services have already been added to the app.</span></span> <span data-ttu-id="32395-630">Такой обходной путь не требуется в ASP.NET Core 3.0.</span><span class="sxs-lookup"><span data-stu-id="32395-630">This workaround won't isn't required in ASP.NET Core 3.0.</span></span>
>
> ```csharp
> private const string HealthCheckServiceAssembly =
>     "Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherHostedService";
>
> services.TryAddEnumerable(
>     ServiceDescriptor.Singleton(typeof(IHostedService),
>         typeof(HealthCheckPublisherOptions).Assembly
>             .GetType(HealthCheckServiceAssembly)));
> ```

> [!NOTE]
> <span data-ttu-id="32395-631">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) содержит издатели для нескольких систем, в том числе [Application Insights](/azure/application-insights/app-insights-overview).</span><span class="sxs-lookup"><span data-stu-id="32395-631">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) includes publishers for several systems, including [Application Insights](/azure/application-insights/app-insights-overview).</span></span>
>
> <span data-ttu-id="32395-632">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) не поддерживается корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="32395-632">[AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) isn't maintained or supported by Microsoft.</span></span>

## <a name="restrict-health-checks-with-mapwhen"></a><span data-ttu-id="32395-633">Ограничение проверок работоспособности с помощью MapWhen</span><span class="sxs-lookup"><span data-stu-id="32395-633">Restrict health checks with MapWhen</span></span>

<span data-ttu-id="32395-634">Используйте <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen*> для условного ветвления конвейера запросов для конечных точек проверки работоспособности.</span><span class="sxs-lookup"><span data-stu-id="32395-634">Use <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen*> to conditionally branch the request pipeline for health check endpoints.</span></span>

<span data-ttu-id="32395-635">В следующем примере `MapWhen` выполняет ветвление конвейера запросов для активации ПО промежуточного слоя для проверки работоспособности, если для конечной точки `api/HealthCheck` поступает запрос GET:</span><span class="sxs-lookup"><span data-stu-id="32395-635">In the following example, `MapWhen` branches the request pipeline to activate Health Checks Middleware if a GET request is received for the `api/HealthCheck` endpoint:</span></span>

```csharp
app.MapWhen(
    context => context.Request.Method == HttpMethod.Get.Method && 
        context.Request.Path.StartsWith("/api/HealthCheck"),
    builder => builder.UseHealthChecks());

app.UseMvc();
```

<span data-ttu-id="32395-636">Для получения дополнительной информации см. <xref:fundamentals/middleware/index#use-run-and-map>.</span><span class="sxs-lookup"><span data-stu-id="32395-636">For more information, see <xref:fundamentals/middleware/index#use-run-and-map>.</span></span>

::: moniker-end
