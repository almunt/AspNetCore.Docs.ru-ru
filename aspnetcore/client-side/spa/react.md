---
title: Использование шаблона проекта React с ASP.NET Core
author: SteveSandersonMS
description: Сведения о начале работы с шаблоном проекта одностраничного приложения (SPA) ASP.NET Core для React и create-react-app.
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 03/07/2019
no-loc:
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: spa/react
ms.openlocfilehash: e9f3bbe2248d1c2fc9030745febb69177c301f7e
ms.sourcegitcommit: 497be502426e9d90bb7d0401b1b9f74b6a384682
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/08/2020
ms.locfileid: "88013129"
---
# <a name="use-the-react-project-template-with-aspnet-core"></a><span data-ttu-id="3d047-103">Использование шаблона проекта React с ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="3d047-103">Use the React project template with ASP.NET Core</span></span>

<span data-ttu-id="3d047-104">Обновленный шаблон проекта React служит удобной отправной точкой для создания приложений ASP.NET Core на основе конвенций React и [create-react-app](https://github.com/facebookincubator/create-react-app) (CRA), позволяющих реализовать многофункциональный пользовательский интерфейс (UI) на стороне клиента.</span><span class="sxs-lookup"><span data-stu-id="3d047-104">The updated React project template provides a convenient starting point for ASP.NET Core apps using React and [create-react-app](https://github.com/facebookincubator/create-react-app) (CRA) conventions to implement a rich, client-side user interface (UI).</span></span>

<span data-ttu-id="3d047-105">Применение этого шаблона эквивалентно одновременному созданию проекта ASP.NET Core в качестве внутреннего API и стандартного проекта CRA в качестве интерфейса пользователя. Но шаблон удобнее тем, что оба интерфейса размещаются в одном проекте приложения, который можно компилировать и публиковать как единое целое.</span><span class="sxs-lookup"><span data-stu-id="3d047-105">The template is equivalent to creating both an ASP.NET Core project to act as an API backend, and a standard CRA React project to act as a UI, but with the convenience of hosting both in a single app project that can be built and published as a single unit.</span></span>

<span data-ttu-id="3d047-106">Шаблон проекта React не предназначен для отрисовки на стороне сервера (SSR).</span><span class="sxs-lookup"><span data-stu-id="3d047-106">The React project template isn't meant for server-side rendering (SSR).</span></span> <span data-ttu-id="3d047-107">Чтобы реализовать SSR с React и Node.js, рекомендуем воспользоваться [Next.js](https://github.com/zeit/next.js/) или [Razzle](https://github.com/jaredpalmer/razzle).</span><span class="sxs-lookup"><span data-stu-id="3d047-107">For SSR with React and Node.js, consider [Next.js](https://github.com/zeit/next.js/) or [Razzle](https://github.com/jaredpalmer/razzle).</span></span>

## <a name="create-a-new-app"></a><span data-ttu-id="3d047-108">Создание нового приложения</span><span class="sxs-lookup"><span data-stu-id="3d047-108">Create a new app</span></span>

<span data-ttu-id="3d047-109">Если установлена ASP.NET Core 2.1, не нужно устанавливать проект шаблона React.</span><span class="sxs-lookup"><span data-stu-id="3d047-109">If you have ASP.NET Core 2.1 installed, there's no need to install the React project template.</span></span>

<span data-ttu-id="3d047-110">Создайте из командной строки новый проект в пустом каталоге с помощью команды `dotnet new react`.</span><span class="sxs-lookup"><span data-stu-id="3d047-110">Create a new project from a command prompt using the command `dotnet new react` in an empty directory.</span></span> <span data-ttu-id="3d047-111">Например, следующие команды позволяют создать приложение в каталоге *my-new-app* и перейти к этому каталогу:</span><span class="sxs-lookup"><span data-stu-id="3d047-111">For example, the following commands create the app in a *my-new-app* directory and switch to that directory:</span></span>

```dotnetcli
dotnet new react -o my-new-app
cd my-new-app
```

<span data-ttu-id="3d047-112">Запустите приложение в Visual Studio или в .NET Core CLI.</span><span class="sxs-lookup"><span data-stu-id="3d047-112">Run the app from either Visual Studio or the .NET Core CLI:</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="3d047-113">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="3d047-113">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="3d047-114">Откройте созданный файл проекта *.csproj* и запустите в нем приложение в обычном режиме.</span><span class="sxs-lookup"><span data-stu-id="3d047-114">Open the generated *.csproj* file, and run the app as normal from there.</span></span>

<span data-ttu-id="3d047-115">В процессе сборки при первом запуске восстанавливаются зависимости npm. Это может занять несколько минут.</span><span class="sxs-lookup"><span data-stu-id="3d047-115">The build process restores npm dependencies on the first run, which can take several minutes.</span></span> <span data-ttu-id="3d047-116">Последующие сборки выполняются гораздо быстрее.</span><span class="sxs-lookup"><span data-stu-id="3d047-116">Subsequent builds are much faster.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="3d047-117">Интерфейс командной строки .NET Core</span><span class="sxs-lookup"><span data-stu-id="3d047-117">.NET Core CLI</span></span>](#tab/netcore-cli)

<span data-ttu-id="3d047-118">Убедитесь, что у вас есть переменная среды с именем `ASPNETCORE_Environment` и значением `Development`.</span><span class="sxs-lookup"><span data-stu-id="3d047-118">Ensure you have an environment variable called `ASPNETCORE_Environment` with value of `Development`.</span></span> <span data-ttu-id="3d047-119">В ОС Windows ее можно создать с помощью команды `SET ASPNETCORE_Environment=Development` (в интерфейсе командной строки, но не в PowerShell).</span><span class="sxs-lookup"><span data-stu-id="3d047-119">On Windows (in non-PowerShell prompts), run `SET ASPNETCORE_Environment=Development`.</span></span> <span data-ttu-id="3d047-120">В ОС Linux или macOS используйте команду `export ASPNETCORE_Environment=Development`.</span><span class="sxs-lookup"><span data-stu-id="3d047-120">On Linux or macOS, run `export ASPNETCORE_Environment=Development`.</span></span>

<span data-ttu-id="3d047-121">Запустите [dotnet build](/dotnet/core/tools/dotnet-build) и убедитесь, что приложение успешно компилируется.</span><span class="sxs-lookup"><span data-stu-id="3d047-121">Run [dotnet build](/dotnet/core/tools/dotnet-build) to verify your app builds correctly.</span></span> <span data-ttu-id="3d047-122">В процессе сборки при первом запуске восстанавливаются зависимости npm. Это может занять несколько минут.</span><span class="sxs-lookup"><span data-stu-id="3d047-122">On the first run, the build process restores npm dependencies, which can take several minutes.</span></span> <span data-ttu-id="3d047-123">Последующие сборки выполняются гораздо быстрее.</span><span class="sxs-lookup"><span data-stu-id="3d047-123">Subsequent builds are much faster.</span></span>

<span data-ttu-id="3d047-124">Запустите [dotnet run](/dotnet/core/tools/dotnet-run) для запуска приложения.</span><span class="sxs-lookup"><span data-stu-id="3d047-124">Run [dotnet run](/dotnet/core/tools/dotnet-run) to start the app.</span></span>

---

<span data-ttu-id="3d047-125">На основе шаблона проекта будут созданы приложения ASP.NET Core и React.</span><span class="sxs-lookup"><span data-stu-id="3d047-125">The project template creates an ASP.NET Core app and a React app.</span></span> <span data-ttu-id="3d047-126">Приложение ASP.NET Core выполняет такие функции, как получение доступа к данным, авторизация и другие процессы на стороне сервера.</span><span class="sxs-lookup"><span data-stu-id="3d047-126">The ASP.NET Core app is intended to be used for data access, authorization, and other server-side concerns.</span></span> <span data-ttu-id="3d047-127">Приложение React, которое находится в подкаталоге *ClientApp*, выполняет все задачи, связанные с пользовательским интерфейсом.</span><span class="sxs-lookup"><span data-stu-id="3d047-127">The React app, residing in the *ClientApp* subdirectory, is intended to be used for all UI concerns.</span></span>

## <a name="add-pages-images-styles-modules-etc"></a><span data-ttu-id="3d047-128">Например, добавление страниц, изображений, стилей, модулей, и т. д</span><span class="sxs-lookup"><span data-stu-id="3d047-128">Add pages, images, styles, modules, etc.</span></span>

<span data-ttu-id="3d047-129">Каталог *ClientApp* содержит стандартное приложение CRA React.</span><span class="sxs-lookup"><span data-stu-id="3d047-129">The *ClientApp* directory is a standard CRA React app.</span></span> <span data-ttu-id="3d047-130">Дополнительные сведения можно найти в [документации по CRA](https://create-react-app.dev/docs/getting-started/).</span><span class="sxs-lookup"><span data-stu-id="3d047-130">See the official [CRA documentation](https://create-react-app.dev/docs/getting-started/) for more information.</span></span>

<span data-ttu-id="3d047-131">Создаваемое с помощью этого шаблона приложение React немного отличается от создаваемого с помощью платформы CRA, но все возможности этих приложений полностью идентичны.</span><span class="sxs-lookup"><span data-stu-id="3d047-131">There are slight differences between the React app created by this template and the one created by CRA itself; however, the app's capabilities are unchanged.</span></span> <span data-ttu-id="3d047-132">Приложение, созданное с помощью шаблона, содержит основанный на [Bootstrap](https://getbootstrap.com/) макет и простой пример маршрутизации.</span><span class="sxs-lookup"><span data-stu-id="3d047-132">The app created by the template contains a [Bootstrap](https://getbootstrap.com/)-based layout and a basic routing example.</span></span>

## <a name="install-npm-packages"></a><span data-ttu-id="3d047-133">Установка пакетов npm</span><span class="sxs-lookup"><span data-stu-id="3d047-133">Install npm packages</span></span>

<span data-ttu-id="3d047-134">Для установки npm-пакетов сторонних разработчиков используйте в командной строке подкаталог *ClientApp*.</span><span class="sxs-lookup"><span data-stu-id="3d047-134">To install third-party npm packages, use a command prompt in the *ClientApp* subdirectory.</span></span> <span data-ttu-id="3d047-135">Пример:</span><span class="sxs-lookup"><span data-stu-id="3d047-135">For example:</span></span>

```console
cd ClientApp
npm install --save <package_name>
```

## <a name="publish-and-deploy"></a><span data-ttu-id="3d047-136">Публикация и развертывание</span><span class="sxs-lookup"><span data-stu-id="3d047-136">Publish and deploy</span></span>

<span data-ttu-id="3d047-137">В процессе разработки приложение запускается в режиме, оптимизированном для удобства разработчиков.</span><span class="sxs-lookup"><span data-stu-id="3d047-137">In development, the app runs in a mode optimized for developer convenience.</span></span> <span data-ttu-id="3d047-138">Например, пакеты JavaScript содержат сопоставления исходного кода (чтобы при отладке можно было проверить исходный код).</span><span class="sxs-lookup"><span data-stu-id="3d047-138">For example, JavaScript bundles include source maps (so that when debugging, you can see your original source code).</span></span> <span data-ttu-id="3d047-139">Приложение отслеживает изменения в файлах JavaScript, HTML и CSS на диске. Когда эти файлы меняются, оно автоматически их повторно компилирует и перезагружает.</span><span class="sxs-lookup"><span data-stu-id="3d047-139">The app watches JavaScript, HTML, and CSS file changes on disk and automatically recompiles and reloads when it sees those files change.</span></span>

<span data-ttu-id="3d047-140">Версию для размещения в рабочей среде следует оптимизировать для повышения производительности.</span><span class="sxs-lookup"><span data-stu-id="3d047-140">In production, serve a version of your app that's optimized for performance.</span></span> <span data-ttu-id="3d047-141">Это выполняется автоматически.</span><span class="sxs-lookup"><span data-stu-id="3d047-141">This is configured to happen automatically.</span></span> <span data-ttu-id="3d047-142">При публикации конфигурация сборки выполняет транскомпиляцию уменьшенной версии клиентского кода.</span><span class="sxs-lookup"><span data-stu-id="3d047-142">When you publish, the build configuration emits a minified, transpiled build of your client-side code.</span></span> <span data-ttu-id="3d047-143">В отличие от сборки для разработки, рабочая сборка не требует установки Node.js на сервер.</span><span class="sxs-lookup"><span data-stu-id="3d047-143">Unlike the development build, the production build doesn't require Node.js to be installed on the server.</span></span>

<span data-ttu-id="3d047-144">Можно использовать стандартные [методы размещения и развертывания ASP.NET Core](xref:host-and-deploy/index).</span><span class="sxs-lookup"><span data-stu-id="3d047-144">You can use standard [ASP.NET Core hosting and deployment methods](xref:host-and-deploy/index).</span></span>

## <a name="run-the-cra-server-independently"></a><span data-ttu-id="3d047-145">Автономный запуск сервера CRA</span><span class="sxs-lookup"><span data-stu-id="3d047-145">Run the CRA server independently</span></span>

<span data-ttu-id="3d047-146">Проект настроен таким образом, что при запуске приложения ASP.NET Core в режиме разработки в фоновом режиме автоматически создается собственный экземпляр сервера разработки CRA.</span><span class="sxs-lookup"><span data-stu-id="3d047-146">The project is configured to start its own instance of the CRA development server in the background when the ASP.NET Core app starts in development mode.</span></span> <span data-ttu-id="3d047-147">Это удобно, так как нет необходимости вручную запускать отдельный сервер.</span><span class="sxs-lookup"><span data-stu-id="3d047-147">This is convenient because it means you don't have to run a separate server manually.</span></span>

<span data-ttu-id="3d047-148">Но у этого стандартного режима есть важный недостаток.</span><span class="sxs-lookup"><span data-stu-id="3d047-148">There's a drawback to this default setup.</span></span> <span data-ttu-id="3d047-149">Сервер CRA перезапускается каждый раз, когда изменяется код на C# и возникает необходимость перезапустить приложение ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="3d047-149">Each time you modify your C# code and your ASP.NET Core app needs to restart, the CRA server restarts.</span></span> <span data-ttu-id="3d047-150">Для создания резервной копии требуется несколько секунд.</span><span class="sxs-lookup"><span data-stu-id="3d047-150">A few seconds are required to start back up.</span></span> <span data-ttu-id="3d047-151">Если вы часто изменяете код C# и не хотите ждать, пока перезапустится сервер CRA, то используйте внешний сервер CRA, который не зависит от процесса ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="3d047-151">If you're making frequent C# code edits and don't want to wait for the CRA server to restart, run the CRA server externally, independently of the ASP.NET Core process.</span></span> <span data-ttu-id="3d047-152">Для этого сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="3d047-152">To do so:</span></span>

1. <span data-ttu-id="3d047-153">Добавьте в подкаталог *ClientApp* файл *ENV* со следующим параметром:</span><span class="sxs-lookup"><span data-stu-id="3d047-153">Add a *.env* file to the *ClientApp* subdirectory with the following setting:</span></span>

    ```
    BROWSER=none
    ```

    <span data-ttu-id="3d047-154">В результате веб-браузер не будет открываться при внешнем запуске сервера CRA.</span><span class="sxs-lookup"><span data-stu-id="3d047-154">This will prevent your web browser from opening when starting the CRA server externally.</span></span>

2. <span data-ttu-id="3d047-155">В командной строке перейдите к подкаталогу *ClientApp* и запустите сервер разработки CRA.</span><span class="sxs-lookup"><span data-stu-id="3d047-155">In a command prompt, switch to the *ClientApp* subdirectory, and launch the CRA development server:</span></span>

    ```console
    cd ClientApp
    npm start
    ```

3. <span data-ttu-id="3d047-156">Измените настройки приложения ASP.NET Core, чтобы оно использовало внешний экземпляр сервера CRA вместо своего встроенного экземпляра.</span><span class="sxs-lookup"><span data-stu-id="3d047-156">Modify your ASP.NET Core app to use the external CRA server instance instead of launching one of its own.</span></span> <span data-ttu-id="3d047-157">В классе *Startup* замените вызов `spa.UseReactDevelopmentServer` следующим кодом:</span><span class="sxs-lookup"><span data-stu-id="3d047-157">In your *Startup* class, replace the `spa.UseReactDevelopmentServer` invocation with the following:</span></span>

    ```csharp
    spa.UseProxyToSpaDevelopmentServer("http://localhost:3000");
    ```

<span data-ttu-id="3d047-158">Теперь приложение ASP.NET Core не будет создавать при запуске сервер CRA.</span><span class="sxs-lookup"><span data-stu-id="3d047-158">When you start your ASP.NET Core app, it won't launch a CRA server.</span></span> <span data-ttu-id="3d047-159">Вместо него оно будет использовать запущенный вручную экземпляр.</span><span class="sxs-lookup"><span data-stu-id="3d047-159">The instance you started manually is used instead.</span></span> <span data-ttu-id="3d047-160">Это позволит быстрее запускать и перезапускать приложение.</span><span class="sxs-lookup"><span data-stu-id="3d047-160">This enables it to start and restart faster.</span></span> <span data-ttu-id="3d047-161">Вам не придется каждый раз ожидать перестроения приложения React.</span><span class="sxs-lookup"><span data-stu-id="3d047-161">It's no longer waiting for your React app to rebuild each time.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="3d047-162">Этот шаблон не поддерживает отрисовку на стороне сервера.</span><span class="sxs-lookup"><span data-stu-id="3d047-162">"Server-side rendering" is not a supported feature of this template.</span></span> <span data-ttu-id="3d047-163">Его цель — обеспечить те же возможности, что и create-react-app.</span><span class="sxs-lookup"><span data-stu-id="3d047-163">Our goal with this template is to meet parity with "create-react-app".</span></span> <span data-ttu-id="3d047-164">Поэтому сценарии и возможности, не включенные в проект create-react-app (в том числе SSR), не поддерживаются. Пользователь может реализовать их самостоятельно в качестве упражнения.</span><span class="sxs-lookup"><span data-stu-id="3d047-164">As such, scenarios and features not included in a "create-react-app" project (such as SSR) are not supported and are left as an exercise for the user.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="3d047-165">Дополнительные ресурсы</span><span class="sxs-lookup"><span data-stu-id="3d047-165">Additional resources</span></span>

* <xref:security/authentication/identity/spa>
