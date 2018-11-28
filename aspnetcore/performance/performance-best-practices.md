---
title: Рекомендации по производительности ASP.NET Core
author: mjrousos
description: Советы для повышения производительности в приложениях ASP.NET Core и как избежать распространенных проблем производительности.
monikerRange: '>= aspnetcore-1.1'
ms.author: riande
ms.date: 11/29/2018
uid: performance/performance-best-practices
ms.openlocfilehash: ced86dbc2d6f40b503493eda122d8977d6df7035
ms.sourcegitcommit: e9b99854b0a8021dafabee0db5e1338067f250a9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/28/2018
ms.locfileid: "52452989"
---
# <a name="aspnet-core-performance-best-practices"></a><span data-ttu-id="af767-103">Рекомендации по производительности ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="af767-103">ASP.NET Core Performance Best Practices</span></span>

<span data-ttu-id="af767-104">По [Майк Роусос](https://github.com/mjrousos)</span><span class="sxs-lookup"><span data-stu-id="af767-104">By [Mike Rousos](https://github.com/mjrousos)</span></span>

<span data-ttu-id="af767-105">В этом разделе приводятся инструкции по производительности рекомендации с помощью ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="af767-105">This topic provides guidelines for performance best practices with ASP.NET Core.</span></span>

<a name="hot"></a>
<span data-ttu-id="af767-106"><!-- TODO review hot code paths is jargon that won't MT (machine translate) and is not well defined for native speakers. --> В этом документе ветви кода определяется как путь кода, который часто называют и где большую часть времени выполнения происходит.</span><span class="sxs-lookup"><span data-stu-id="af767-106"><!-- TODO review hot code paths is jargon that won't MT (machine translate) and is not well defined for native speakers. --> In this document, a hot code path is defined as a code path that is frequently called and where much of the execution time occurs.</span></span> <span data-ttu-id="af767-107">"Горячий" путей обычно ограничивают приложения масштабирования и производительности.</span><span class="sxs-lookup"><span data-stu-id="af767-107">Hot code paths typically limit app scale-out and performance.</span></span>

## <a name="cache-aggressively"></a><span data-ttu-id="af767-108">Кэшировать агрессивно</span><span class="sxs-lookup"><span data-stu-id="af767-108">Cache aggressively</span></span>

<span data-ttu-id="af767-109">Кэширование обсуждается в нескольких частях этого документа.</span><span class="sxs-lookup"><span data-stu-id="af767-109">Caching is discussed in several parts of this document.</span></span> <span data-ttu-id="af767-110">Дополнительные сведения см. в разделе [кэшировать ответы в ASP.NET Core](xref:performance/caching/index).</span><span class="sxs-lookup"><span data-stu-id="af767-110">For more information, see [Cache responses in ASP.NET Core](xref:performance/caching/index).</span></span>

## <a name="avoid-blocking-calls"></a><span data-ttu-id="af767-111">Избегайте блокирующих вызовов</span><span class="sxs-lookup"><span data-stu-id="af767-111">Avoid blocking calls</span></span>

<span data-ttu-id="af767-112">Приложения ASP.NET Core должно уметь обрабатывать множество запросов одновременно.</span><span class="sxs-lookup"><span data-stu-id="af767-112">ASP.NET Core apps should be designed to process many requests simultaneously.</span></span> <span data-ttu-id="af767-113">Асинхронные интерфейсы API позволяют небольшой пул потоков для обработки тысяч одновременных запросов, не ожидая на блокирования вызовов.</span><span class="sxs-lookup"><span data-stu-id="af767-113">Asynchronous APIs allow a small pool of threads to handle thousands of concurrent requests by not waiting on blocking calls.</span></span> <span data-ttu-id="af767-114">Вместо того чтобы ожидание завершения выполнения задачи синхронной выполняющейся длительное время, поток может работать на другой запрос.</span><span class="sxs-lookup"><span data-stu-id="af767-114">Rather than waiting on a long-running synchronous task to complete, the thread can work on another request.</span></span>

<span data-ttu-id="af767-115">Распространенные проблемы с производительностью в приложениях ASP.NET Core блокирует вызовы, которые могут выполняться асинхронно.</span><span class="sxs-lookup"><span data-stu-id="af767-115">A common performance problem in ASP.NET Core apps is blocking calls that could be asynchronous.</span></span> <span data-ttu-id="af767-116">Многие синхронных вызовов блокировки приводит к [нехватка потоков пула](https://blogs.msdn.microsoft.com/vancem/2018/10/16/diagnosing-net-core-threadpool-starvation-with-perfview-why-my-service-is-not-saturating-all-cores-or-seems-to-stall/) и снижения времени отклика.</span><span class="sxs-lookup"><span data-stu-id="af767-116">Many synchronous blocking calls leads to [Thread Pool starvation](https://blogs.msdn.microsoft.com/vancem/2018/10/16/diagnosing-net-core-threadpool-starvation-with-perfview-why-my-service-is-not-saturating-all-cores-or-seems-to-stall/) and degrading response times.</span></span>

<span data-ttu-id="af767-117">**Не**:</span><span class="sxs-lookup"><span data-stu-id="af767-117">**Do not**:</span></span>

* <span data-ttu-id="af767-118">Блокировать асинхронное выполнение путем вызова [Task.Wait](/dotnet/api/system.threading.tasks.task.wait) или [Task.Result](/dotnet/api/system.threading.tasks.task-1.result).</span><span class="sxs-lookup"><span data-stu-id="af767-118">Block asynchronous execution by calling [Task.Wait](/dotnet/api/system.threading.tasks.task.wait) or [Task.Result](/dotnet/api/system.threading.tasks.task-1.result).</span></span>
* <span data-ttu-id="af767-119">Блокировки в общих путей кода.</span><span class="sxs-lookup"><span data-stu-id="af767-119">Acquire locks in common code paths.</span></span> <span data-ttu-id="af767-120">Приложения ASP.NET Core — это самых производительных, когда создано для запуска кода в параллельном режиме.</span><span class="sxs-lookup"><span data-stu-id="af767-120">ASP.NET Core apps are most performant when architected to run code in parallel.</span></span>

<span data-ttu-id="af767-121">**Сделать**:</span><span class="sxs-lookup"><span data-stu-id="af767-121">**Do**:</span></span>

* <span data-ttu-id="af767-122">Сделать [горячего пути кода](#hot) асинхронной.</span><span class="sxs-lookup"><span data-stu-id="af767-122">Make [hot code paths](#hot) asynchronous.</span></span>
* <span data-ttu-id="af767-123">Асинхронный вызов доступа к данным и длительных операций API-интерфейсы.</span><span class="sxs-lookup"><span data-stu-id="af767-123">Call data access and long-running operations APIs asynchronously.</span></span>
* <span data-ttu-id="af767-124">Сделать контроллер/Razor действия страницы асинхронным.</span><span class="sxs-lookup"><span data-stu-id="af767-124">Make controller/Razor Page actions asynchronous.</span></span> <span data-ttu-id="af767-125">Весь стек вызовов должен выполняться асинхронно, чтобы использовать преимущества [async/await](/dotnet/csharp/programming-guide/concepts/async/) шаблонов.</span><span class="sxs-lookup"><span data-stu-id="af767-125">The entire call stack needs to be asynchronous in order to benefit from [async/await](/dotnet/csharp/programming-guide/concepts/async/) patterns.</span></span>

<span data-ttu-id="af767-126">Профилировщик, такие как [PerfView](https://github.com/Microsoft/perfview) может использоваться для поиска для потоков часто, добавляемый [пула потоков](/windows/desktop/procthread/thread-pool).</span><span class="sxs-lookup"><span data-stu-id="af767-126">A profiler like [PerfView](https://github.com/Microsoft/perfview) can be used to look for threads frequently being added to the [Thread Pool](/windows/desktop/procthread/thread-pool).</span></span> <span data-ttu-id="af767-127">`Microsoft-Windows-DotNETRuntime/ThreadPoolWorkerThread/Start` Указывает поток, добавляемые в пул потоков.</span><span class="sxs-lookup"><span data-stu-id="af767-127">The `Microsoft-Windows-DotNETRuntime/ThreadPoolWorkerThread/Start` event indicates a thread being added to the thread pool.</span></span> <!--  For more information, see [async guidance docs](TBD-Link_To_Davifowl_Doc  -->

## <a name="minimize-large-object-allocations"></a><span data-ttu-id="af767-128">Свести к минимуму распределения больших объектов</span><span class="sxs-lookup"><span data-stu-id="af767-128">Minimize large object allocations</span></span>

<span data-ttu-id="af767-129"><!-- TODO review Bill - replaced original .NET language below with .NET Core since this targets .NET Core --> [Сборщик мусора .NET Core](/dotnet/standard/garbage-collection/) управляет выделением и освобождением памяти автоматически в приложениях ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="af767-129"><!-- TODO review Bill - replaced original .NET language below with .NET Core since this targets .NET Core --> The [.NET Core garbage collector](/dotnet/standard/garbage-collection/) manages allocation and release of memory automatically in ASP.NET Core apps.</span></span> <span data-ttu-id="af767-130">Автоматическая сборка мусора обычно означает, что разработчикам не нужно беспокоиться о способе и времени память освобождается.</span><span class="sxs-lookup"><span data-stu-id="af767-130">Automatic garbage collection generally means that developers don't need to worry about how or when memory is freed.</span></span> <span data-ttu-id="af767-131">Тем не менее, очистка неиспользуемых объектов требует затрат Процессорного времени, поэтому разработчики должны свести к минимуму выделения объектов в [горячего пути кода](#hot).</span><span class="sxs-lookup"><span data-stu-id="af767-131">However, cleaning up unreferenced objects takes CPU time, so developers should minimize allocating objects in [hot code paths](#hot).</span></span> <span data-ttu-id="af767-132">Сборка мусора является особенно сильно на больших объектов (в байтах > 85 КБ).</span><span class="sxs-lookup"><span data-stu-id="af767-132">Garbage collection is especially expensive on large objects (> 85 K bytes).</span></span> <span data-ttu-id="af767-133">Большие объекты хранятся на [кучи больших объектов](/dotnet/standard/garbage-collection/large-object-heap) и требуют полной (поколение 2) сбора мусора для очистки.</span><span class="sxs-lookup"><span data-stu-id="af767-133">Large objects are stored on the [large object heap](/dotnet/standard/garbage-collection/large-object-heap) and require a full (generation 2) garbage collection to clean up.</span></span> <span data-ttu-id="af767-134">В отличие от поколения 0 и 1 поколения поколения 2 требуется временно приостановить выполнение приложения.</span><span class="sxs-lookup"><span data-stu-id="af767-134">Unlike generation 0 and generation 1 collections, a generation 2 collection requires app execution to be temporarily suspended.</span></span> <span data-ttu-id="af767-135">Частые о выделении и освобождении больших объектов может привести к нестабильной работе.</span><span class="sxs-lookup"><span data-stu-id="af767-135">Frequent allocation and de-allocation of large objects can cause inconsistent performance.</span></span>

<span data-ttu-id="af767-136">Рекомендуемые действия.</span><span class="sxs-lookup"><span data-stu-id="af767-136">Recommendations:</span></span>

* <span data-ttu-id="af767-137">**Сделать** предусмотреть кэширование больших объектов, которые часто используются.</span><span class="sxs-lookup"><span data-stu-id="af767-137">**Do** consider caching large objects that are frequently used.</span></span> <span data-ttu-id="af767-138">Кэширование больших объектов не позволяет дорогостоящие выделения.</span><span class="sxs-lookup"><span data-stu-id="af767-138">Caching large objects prevents expensive allocations.</span></span>
* <span data-ttu-id="af767-139">**Сделать** пула буферов с помощью [ `ArrayPool<T>` ](/dotnet/api/system.buffers.arraypool-1) для хранения больших массивов.</span><span class="sxs-lookup"><span data-stu-id="af767-139">**Do** pool buffers by using an [`ArrayPool<T>`](/dotnet/api/system.buffers.arraypool-1) to store large arrays.</span></span>
* <span data-ttu-id="af767-140">**Не** распределить коротким сроком существования, многие большие объекты на [горячего пути кода](#hot).</span><span class="sxs-lookup"><span data-stu-id="af767-140">**Do not** allocate many, short-lived large objects on [hot code paths](#hot).</span></span>

<span data-ttu-id="af767-141">Проблемы с памятью, как приведенный выше можно диагностировать, просмотрев статистика сборки мусора (GC) коллекции в [PerfView](https://github.com/Microsoft/perfview) и изучив:</span><span class="sxs-lookup"><span data-stu-id="af767-141">Memory issues like the preceding can be diagnosed by reviewing garbage collection (GC) stats in [PerfView](https://github.com/Microsoft/perfview) and examining:</span></span>

* <span data-ttu-id="af767-142">Время приостановки сбора мусора.</span><span class="sxs-lookup"><span data-stu-id="af767-142">Garbage collection pause time.</span></span>
* <span data-ttu-id="af767-143">Какой процент времени процессора, затраченное на сборку мусора.</span><span class="sxs-lookup"><span data-stu-id="af767-143">What percentage of the processor time is spent in garbage collection.</span></span>
* <span data-ttu-id="af767-144">Количество сборок мусора — поколение 0, 1 и 2.</span><span class="sxs-lookup"><span data-stu-id="af767-144">How many garbage collections are generation 0, 1, and 2.</span></span>

<span data-ttu-id="af767-145">Дополнительные сведения см. в разделе [мусора и производительность](/dotnet/standard/garbage-collection/performance).</span><span class="sxs-lookup"><span data-stu-id="af767-145">For more information, see [Garbage Collection and Performance](/dotnet/standard/garbage-collection/performance).</span></span>

## <a name="optimize-data-access"></a><span data-ttu-id="af767-146">Оптимизация доступа к данным</span><span class="sxs-lookup"><span data-stu-id="af767-146">Optimize Data Access</span></span>

<span data-ttu-id="af767-147">Взаимодействие с хранилищем данных и других удаленных служб зачастую в самые медленные части приложения ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="af767-147">Interactions with a data store or other remote services are often the slowest part of an ASP.NET Core app.</span></span> <span data-ttu-id="af767-148">Чтение и запись данных эффективно крайне важно для высокой производительности.</span><span class="sxs-lookup"><span data-stu-id="af767-148">Reading and writing data efficiently is critical for good performance.</span></span>

<span data-ttu-id="af767-149">Рекомендуемые действия.</span><span class="sxs-lookup"><span data-stu-id="af767-149">Recommendations:</span></span>

* <span data-ttu-id="af767-150">**Сделать** асинхронный вызов доступа ко всем данным API-интерфейсы.</span><span class="sxs-lookup"><span data-stu-id="af767-150">**Do** call all data access APIs asynchronously.</span></span>
* <span data-ttu-id="af767-151">**Не** получить больше данных, чем необходимо.</span><span class="sxs-lookup"><span data-stu-id="af767-151">**Do not** retrieve more data than is necessary.</span></span> <span data-ttu-id="af767-152">Написать запросы для получения только данные, необходимые для текущего HTTP-запроса.</span><span class="sxs-lookup"><span data-stu-id="af767-152">Write queries to return just the data that is necessary for the current HTTP request.</span></span>
* <span data-ttu-id="af767-153">**Сделать** рассмотрим кэширование часто запрашиваемых данных, полученных из базы данных или удаленной службы, если это допустимо для данных, которые требуется немного устарела.</span><span class="sxs-lookup"><span data-stu-id="af767-153">**Do** consider caching frequently accessed data retrieved from a database or remote service if it is acceptable for the data to be slightly out-of-date.</span></span> <span data-ttu-id="af767-154">В зависимости от сценария, можно использовать [MemoryCache](xref:performance/caching/memory) или [DistributedCache](xref:performance/caching/distributed).</span><span class="sxs-lookup"><span data-stu-id="af767-154">Depending on the scenario, you might use a [MemoryCache](xref:performance/caching/memory) or a [DistributedCache](xref:performance/caching/distributed).</span></span> <span data-ttu-id="af767-155">Дополнительные сведения см. в разделе [кэшировать ответы в ASP.NET Core](xref:performance/caching/index).</span><span class="sxs-lookup"><span data-stu-id="af767-155">For more information, see [Cache responses in ASP.NET Core](xref:performance/caching/index).</span></span>
* <span data-ttu-id="af767-156">Свести к минимуму сетевой циклами.</span><span class="sxs-lookup"><span data-stu-id="af767-156">Minimize network round trips.</span></span> <span data-ttu-id="af767-157">Целью является для получения всех данных, которые потребуются в одном вызове, а не несколько вызовов.</span><span class="sxs-lookup"><span data-stu-id="af767-157">The goal is to retrieve all the data that will be needed in a single call rather than several calls.</span></span>
* <span data-ttu-id="af767-158">**Сделать** использовать [Отключение отслеживания запросов](/ef/core/querying/tracking#no-tracking-queries) в Entity Framework Core, при доступе к данным в целях только для чтения.</span><span class="sxs-lookup"><span data-stu-id="af767-158">**Do** use [no-tracking queries](/ef/core/querying/tracking#no-tracking-queries) in Entity Framework Core when accessing data for read-only purposes.</span></span> <span data-ttu-id="af767-159">EF Core может возвращать результаты Отключение отслеживания запросов более эффективно.</span><span class="sxs-lookup"><span data-stu-id="af767-159">EF Core can return the results of no-tracking queries more efficiently.</span></span>
* <span data-ttu-id="af767-160">**Сделать** фильтра и объединение запросов LINQ (с `.Where`, `.Select`, или `.Sum` инструкций, например), которая выполняется в базе данных.</span><span class="sxs-lookup"><span data-stu-id="af767-160">**Do** filter and aggregate LINQ queries (with `.Where`, `.Select`, or `.Sum` statements, for example) so that the filtering is done by the database.</span></span>
* <span data-ttu-id="af767-161">**Сделать** учитывать, что EF Core разрешает некоторые операторы запроса на клиенте, что может привести к выполнению неэффективным.</span><span class="sxs-lookup"><span data-stu-id="af767-161">**Do** consider that EF Core resolves some query operators on the client, which may lead to inefficient query execution.</span></span> <span data-ttu-id="af767-162">Дополнительные сведения см. в разделе [проблем с производительностью оценки клиента](/ef/core/querying/client-eval#client-evaluation-performance-issues)</span><span class="sxs-lookup"><span data-stu-id="af767-162">For more information, see [Client evaluation performance issues](/ef/core/querying/client-eval#client-evaluation-performance-issues)</span></span>
* <span data-ttu-id="af767-163">**Не** использовать запросы проекции для коллекций, что может привести выполнение «N + 1» SQL-запросов.</span><span class="sxs-lookup"><span data-stu-id="af767-163">**Do not** use projection queries on collections, which can result in executing "N + 1" SQL queries.</span></span> <span data-ttu-id="af767-164">Дополнительные сведения см. в разделе [оптимизация коррелированных вложенных запросов](/ef/core/what-is-new/ef-core-2.1#optimization-of-correlated-subqueries).</span><span class="sxs-lookup"><span data-stu-id="af767-164">For more information, see [Optimization of correlated subqueries](/ef/core/what-is-new/ef-core-2.1#optimization-of-correlated-subqueries).</span></span>

<span data-ttu-id="af767-165">См. в разделе [EF высокопроизводительных](/ef/core/what-is-new/ef-core-2.0#explicitly-compiled-queries) для подходы, которые могут повысить производительность в крупномасштабных приложениях:</span><span class="sxs-lookup"><span data-stu-id="af767-165">See [EF High Performance](/ef/core/what-is-new/ef-core-2.0#explicitly-compiled-queries) for approaches that may improve performance in high-scale apps:</span></span>

* [<span data-ttu-id="af767-166">Создание пулов DbContext</span><span class="sxs-lookup"><span data-stu-id="af767-166">DbContext pooling</span></span>](/ef/core/what-is-new/ef-core-2.0#dbcontext-pooling)
* [<span data-ttu-id="af767-167">Явным образом скомпилированные запросы</span><span class="sxs-lookup"><span data-stu-id="af767-167">Explicitly compiled queries</span></span>](/ef/core/what-is-new/ef-core-2.0#explicitly-compiled-queries)

<span data-ttu-id="af767-168">Мы рекомендуем оценить результаты предыдущего подхода высокой производительности перед фиксацией в базе данных кода.</span><span class="sxs-lookup"><span data-stu-id="af767-168">We recommend you measure the impact of the preceding high-performance approaches before committing to your code base.</span></span> <span data-ttu-id="af767-169">Дополнительные сложности скомпилированных запросов может не оправдать повышение производительности.</span><span class="sxs-lookup"><span data-stu-id="af767-169">The additional complexity of compiled queries may not justify the performance improvement.</span></span>

<span data-ttu-id="af767-170">Запрос, могут быть обнаружены проблемы, просмотрев время, затраченное на доступ к данным с помощью [Application Insights](/azure/application-insights/app-insights-overview) или с помощью средств профилирования.</span><span class="sxs-lookup"><span data-stu-id="af767-170">Query issues can be detected by reviewing time spent accessing data with [Application Insights](/azure/application-insights/app-insights-overview) or with profiling tools.</span></span> <span data-ttu-id="af767-171">Большинство баз данных также доступна статистика о частых запросов.</span><span class="sxs-lookup"><span data-stu-id="af767-171">Most databases also make statistics available concerning frequently executed queries.</span></span>

## <a name="pool-http-connections-with-httpclientfactory"></a><span data-ttu-id="af767-172">Пул HTTP-подключений с HttpClientFactory</span><span class="sxs-lookup"><span data-stu-id="af767-172">Pool HTTP connections with HttpClientFactory</span></span>

<span data-ttu-id="af767-173">Несмотря на то что [HttpClient](/dotnet/api/system.net.http.httpclient?view=netstandard-2.0) реализует `IDisposable` интерфейс, она создана для повторного использования.</span><span class="sxs-lookup"><span data-stu-id="af767-173">Although [HttpClient](/dotnet/api/system.net.http.httpclient?view=netstandard-2.0) implements the `IDisposable` interface, it's meant to be reused.</span></span> <span data-ttu-id="af767-174">Закрыто `HttpClient` экземпляров не закрывайте сокетов в `TIME_WAIT` состояния на короткое время.</span><span class="sxs-lookup"><span data-stu-id="af767-174">Closed `HttpClient` instances leave sockets open in the `TIME_WAIT` state for a short period of time.</span></span> <span data-ttu-id="af767-175">Следовательно Если путь кода, который создает и удаляет `HttpClient` объекты часто используется, приложение может исчерпать все доступные сокетов.</span><span class="sxs-lookup"><span data-stu-id="af767-175">Consequently, if a code path that creates and disposes of `HttpClient` objects is frequently used, the app may exhaust available sockets.</span></span> <span data-ttu-id="af767-176">[HttpClientFactory](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests) появилась в ASP.NET Core 2.1 в качестве решения этой проблемы.</span><span class="sxs-lookup"><span data-stu-id="af767-176">[HttpClientFactory](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests) was introduced in ASP.NET Core 2.1 as a solution to this problem.</span></span> <span data-ttu-id="af767-177">Он обрабатывает регулирования количества запросов HTTP-подключения для оптимизации производительности и надежности.</span><span class="sxs-lookup"><span data-stu-id="af767-177">It handles pooling HTTP connections to optimize performance and reliability.</span></span>

<span data-ttu-id="af767-178">Рекомендуемые действия.</span><span class="sxs-lookup"><span data-stu-id="af767-178">Recommendations:</span></span>

* <span data-ttu-id="af767-179">**Не** создания и удаления объекта `HttpClient` экземпляры напрямую.</span><span class="sxs-lookup"><span data-stu-id="af767-179">**Do not** create and dispose of `HttpClient` instances directly.</span></span>
* <span data-ttu-id="af767-180">**Сделать** использовать [HttpClientFactory](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests) извлекаемого `HttpClient` экземпляров.</span><span class="sxs-lookup"><span data-stu-id="af767-180">**Do** use [HttpClientFactory](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests) to retrieve `HttpClient` instances.</span></span> <span data-ttu-id="af767-181">Дополнительные сведения см. в разделе [HttpClientFactory использования для реализации устойчивой HTTP-запросы](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests).</span><span class="sxs-lookup"><span data-stu-id="af767-181">For more information, see [Use HttpClientFactory to implement resilient HTTP requests](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests).</span></span>

## <a name="keep-common-code-paths-fast"></a><span data-ttu-id="af767-182">Хранить быстро общих путей кода</span><span class="sxs-lookup"><span data-stu-id="af767-182">Keep common code paths fast</span></span>

<span data-ttu-id="af767-183">Требуется, весь код требуется высокая скорость работы, но часто вызываемой путей наиболее важны для оптимизации:</span><span class="sxs-lookup"><span data-stu-id="af767-183">You want all of your code to be fast, but frequently called code paths are the most critical to optimize:</span></span>

* <span data-ttu-id="af767-184">Компоненты промежуточного слоя в конвейере обработки запросов приложения, особенно по промежуточного слоя для запуска на раннем этапе конвейера.</span><span class="sxs-lookup"><span data-stu-id="af767-184">Middleware components in the app's request processing pipeline, especially middleware run early in the pipeline.</span></span> <span data-ttu-id="af767-185">Эти компоненты имеют значительное влияние на производительность.</span><span class="sxs-lookup"><span data-stu-id="af767-185">These components have a large impact on performance.</span></span>
* <span data-ttu-id="af767-186">Код, выполняемый для каждого запроса несколько раз или на один запрос.</span><span class="sxs-lookup"><span data-stu-id="af767-186">Code that is executed for every request or multiple times per request.</span></span> <span data-ttu-id="af767-187">Например настраиваемое протоколирование, обработчики авторизации или инициализации временных служб.</span><span class="sxs-lookup"><span data-stu-id="af767-187">For example, custom logging, authorization handlers, or initialization of transient services.</span></span>

<span data-ttu-id="af767-188">Рекомендуемые действия.</span><span class="sxs-lookup"><span data-stu-id="af767-188">Recommendations:</span></span>

* <span data-ttu-id="af767-189">**Не** использовать компоненты пользовательского по промежуточного слоя с длительно выполняемых задач.</span><span class="sxs-lookup"><span data-stu-id="af767-189">**Do not** use custom middleware components with long-running tasks.</span></span>
* <span data-ttu-id="af767-190">**Сделать** использовать средства профилирования производительности (такие как [средств диагностики Visual Studio](/visualstudio/profiling/profiling-feature-tour) или [PerfView](https://github.com/Microsoft/perfview)) для идентификации [горячего пути кода](#hot).</span><span class="sxs-lookup"><span data-stu-id="af767-190">**Do** use performance profiling tools (like [Visual Studio Diagnostic Tools](/visualstudio/profiling/profiling-feature-tour) or [PerfView](https://github.com/Microsoft/perfview)) to identify [hot code paths](#hot).</span></span>

## <a name="complete-long-running-tasks-outside-of-http-requests"></a><span data-ttu-id="af767-191">Завершить долго выполняющихся задач вне HTTP-запросов</span><span class="sxs-lookup"><span data-stu-id="af767-191">Complete long-running Tasks outside of HTTP requests</span></span>

<span data-ttu-id="af767-192">Большинство запросов для приложения ASP.NET Core могут быть обработаны контроллера или модели страницы, вызывая необходимые службы и возвращает ответ HTTP.</span><span class="sxs-lookup"><span data-stu-id="af767-192">Most requests to an ASP.NET Core app can be handled by a controller or page model calling necessary services and returning an HTTP response.</span></span> <span data-ttu-id="af767-193">Для некоторых запросов, включающих длительно выполняемых задач лучше сделать асинхронный процесс весь запрос ответ.</span><span class="sxs-lookup"><span data-stu-id="af767-193">For some requests that involve long-running tasks, it's better to make the entire request-response process asynchronous.</span></span>

<span data-ttu-id="af767-194">Рекомендуемые действия.</span><span class="sxs-lookup"><span data-stu-id="af767-194">Recommendations:</span></span>

* <span data-ttu-id="af767-195">**Не** ожидания длительные задачи, выполняемые как часть обычных обработки HTTP-запроса.</span><span class="sxs-lookup"><span data-stu-id="af767-195">**Do not** wait for long-running tasks to complete as part of ordinary HTTP request processing.</span></span>
* <span data-ttu-id="af767-196">**Сделать** рассмотрите возможность обработки длительных запросов с [фоновых служб](/aspnet/core/fundamentals/host/hosted-services) или вне процесса с [Azure функция](/azure/azure-functions/).</span><span class="sxs-lookup"><span data-stu-id="af767-196">**Do** consider handling long-running requests with [background services](/aspnet/core/fundamentals/host/hosted-services) or out of process with an [Azure Function](/azure/azure-functions/).</span></span> <span data-ttu-id="af767-197">Завершение работы вне процесса особенно удобна для задач, интенсивно использующие ЦП.</span><span class="sxs-lookup"><span data-stu-id="af767-197">Completing work out-of-process is especially beneficial for CPU-intensive tasks.</span></span>
* <span data-ttu-id="af767-198">**Сделать** вариантов взаимодействия в режиме реального времени, как использовать [SignalR](xref:signalr/introduction) для асинхронного взаимодействия с клиентами.</span><span class="sxs-lookup"><span data-stu-id="af767-198">**Do** use real-time communication options like [SignalR](xref:signalr/introduction) to communicate with clients asynchronously.</span></span>

## <a name="minify-client-assets"></a><span data-ttu-id="af767-199">Уменьшить размер активы клиента</span><span class="sxs-lookup"><span data-stu-id="af767-199">Minify client assets</span></span>

<span data-ttu-id="af767-200">Приложений ASP.NET Core с помощью сложных интерфейсов часто служат многих JavaScript, CSS или файлы изображений.</span><span class="sxs-lookup"><span data-stu-id="af767-200">ASP.NET Core apps with complex front-ends frequently serve many JavaScript, CSS, or image files.</span></span> <span data-ttu-id="af767-201">Можно повысить производительность запросов начальной загрузки:</span><span class="sxs-lookup"><span data-stu-id="af767-201">Performance of initial load requests can be improved by:</span></span>

* <span data-ttu-id="af767-202">Объединение, который объединяет несколько файлов в один.</span><span class="sxs-lookup"><span data-stu-id="af767-202">Bundling, which combines multiple files into one.</span></span>
* <span data-ttu-id="af767-203">Уменьшения, что уменьшает размер файлов.</span><span class="sxs-lookup"><span data-stu-id="af767-203">Minifying, which reduces the size of files by.</span></span>

<span data-ttu-id="af767-204">Рекомендуемые действия.</span><span class="sxs-lookup"><span data-stu-id="af767-204">Recommendations:</span></span>

* <span data-ttu-id="af767-205">**Сделать** использовать ASP.NET Core [встроенную поддержку](xref:client-side/bundling-and-minification) для объединение и Минификация активов клиента.</span><span class="sxs-lookup"><span data-stu-id="af767-205">**Do** use ASP.NET Core's [built-in support](xref:client-side/bundling-and-minification) for bundling and minifying client assets.</span></span>
* <span data-ttu-id="af767-206">**Сделать** рассмотрим другие сторонние средства, такие как [Gulp](uid:client-side/bundling-and-minification#consume-bundleconfigjson-from-gulp) или [Webpack](https://webpack.js.org/) для более сложных управление ресурсами клиента.</span><span class="sxs-lookup"><span data-stu-id="af767-206">**Do** consider other third-party tools like [Gulp](uid:client-side/bundling-and-minification#consume-bundleconfigjson-from-gulp) or [Webpack](https://webpack.js.org/) for more complex client asset management.</span></span>

## <a name="use-the-latest-aspnet-core-release"></a><span data-ttu-id="af767-207">Используйте последний выпуск ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="af767-207">Use the latest ASP.NET Core release</span></span>

<span data-ttu-id="af767-208">Каждый новый выпуск ASP.NET включает в себя повышение производительности.</span><span class="sxs-lookup"><span data-stu-id="af767-208">Each new release of ASP.NET includes performance improvements.</span></span> <span data-ttu-id="af767-209">Оптимизации в .NET Core и ASP.NET Core означает, что более новые версии будут более эффективны, чем предыдущие версии.</span><span class="sxs-lookup"><span data-stu-id="af767-209">Optimizations in .NET Core and ASP.NET Core mean that newer versions will outperform older versions.</span></span> <span data-ttu-id="af767-210">Например, .NET Core 2.1 добавлена поддержка для скомпилированные регулярные выражения и использует преимущества [ `Span<T>` ](https://msdn.microsoft.com/magazine/mt814808.aspx).</span><span class="sxs-lookup"><span data-stu-id="af767-210">For example, .NET Core 2.1 added support for compiled regular expressions and benefitted from [`Span<T>`](https://msdn.microsoft.com/magazine/mt814808.aspx).</span></span> <span data-ttu-id="af767-211">ASP.NET Core 2.2 добавлена поддержка HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="af767-211">ASP.NET Core 2.2 added support for HTTP/2.</span></span> <span data-ttu-id="af767-212">Если важна производительность, рассмотрите возможность обновления до последней версии ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="af767-212">If performance is a priority, consider upgrading to the most current version of ASP.NET Core.</span></span>

<!-- TODO review link and taking advantage of new [performance features](#TBD)
Maybe skip this TBD link as each version will have perf improvements -->

## <a name="minimize-exceptions"></a><span data-ttu-id="af767-213">Свести к минимуму исключения</span><span class="sxs-lookup"><span data-stu-id="af767-213">Minimize exceptions</span></span>

<span data-ttu-id="af767-214">Исключения должны быть редко.</span><span class="sxs-lookup"><span data-stu-id="af767-214">Exceptions should be rare.</span></span> <span data-ttu-id="af767-215">Создание и перехват исключений работает медленно относительно других шаблонов потока кода.</span><span class="sxs-lookup"><span data-stu-id="af767-215">Throwing and catching exceptions is slow relative to other code flow patterns.</span></span> <span data-ttu-id="af767-216">По этой причине исключения не должен использоваться для управления выполнением программы в обычном режиме.</span><span class="sxs-lookup"><span data-stu-id="af767-216">Because of this, exceptions should not be used to control normal program flow.</span></span>

<span data-ttu-id="af767-217">Рекомендуемые действия.</span><span class="sxs-lookup"><span data-stu-id="af767-217">Recommendations:</span></span>

* <span data-ttu-id="af767-218">**Не** используйте Создание или перехват исключений с точки зрения нормального выполнения программы, особенно в пути кода "Горячий".</span><span class="sxs-lookup"><span data-stu-id="af767-218">**Do not** use throwing or catching exceptions as a means of normal program flow, especially in hot code paths.</span></span>
* <span data-ttu-id="af767-219">**Сделать** включить логику в приложении, чтобы обнаружить и обработать условия возникновения исключения.</span><span class="sxs-lookup"><span data-stu-id="af767-219">**Do** include logic in the app to detect and handle conditions that would cause an exception.</span></span>
* <span data-ttu-id="af767-220">**Сделать** выдать или перехватить исключения для нестандартных или непредвиденных условий.</span><span class="sxs-lookup"><span data-stu-id="af767-220">**Do** throw or catch exceptions for unusual or unexpected conditions.</span></span>

<span data-ttu-id="af767-221">Средства диагностики приложений (например, Application Insights) может помочь определить общие исключения в приложении, что может повлиять на производительность.</span><span class="sxs-lookup"><span data-stu-id="af767-221">App diagnostic tools (like Application Insights) can help to identify common exceptions in an application which may affect performance.</span></span>