---
title: Вызов методов .NET из функций JavaScript в ASP.NET Core Blazor
author: guardrex
description: Узнайте, как вызывать методы .NET из функций JavaScript в приложениях Blazor.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/19/2020
no-loc:
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/call-dotnet-from-javascript
ms.openlocfilehash: 91f2aa893c06728b4b71d010241a2cb5a307ae0b
ms.sourcegitcommit: d65a027e78bf0b83727f975235a18863e685d902
ms.contentlocale: ru-RU
ms.lasthandoff: 06/26/2020
ms.locfileid: "85400197"
---
# <a name="call-net-methods-from-javascript-functions-in-aspnet-core-blazor"></a><span data-ttu-id="0aba8-103">Вызов методов .NET из функций JavaScript в ASP.NET Core Blazor</span><span class="sxs-lookup"><span data-stu-id="0aba8-103">Call .NET methods from JavaScript functions in ASP.NET Core Blazor</span></span>

<span data-ttu-id="0aba8-104">Авторы: [Хавьер Кальварро Нельсон](https://github.com/javiercn) (Javier Calvarro Nelson), [Дэниэл Рот](https://github.com/danroth27) (Daniel Roth) и [Шашикант Рудравади](http://wisne.co) (Shashikant Rudrawadi) и [Люк Латэм](https://github.com/guardrex) (Luke Latham)</span><span class="sxs-lookup"><span data-stu-id="0aba8-104">By [Javier Calvarro Nelson](https://github.com/javiercn), [Daniel Roth](https://github.com/danroth27), [Shashikant Rudrawadi](http://wisne.co), and [Luke Latham](https://github.com/guardrex)</span></span>

<span data-ttu-id="0aba8-105">Приложение Blazor может вызывать функции JavaScript из методов .NET и методы .NET из функций JavaScript.</span><span class="sxs-lookup"><span data-stu-id="0aba8-105">A Blazor app can invoke JavaScript functions from .NET methods and .NET methods from JavaScript functions.</span></span> <span data-ttu-id="0aba8-106">Такой подход называется *взаимодействием с JavaScript* (*JS*).</span><span class="sxs-lookup"><span data-stu-id="0aba8-106">These scenarios are called *JavaScript interoperability* (*JS interop*).</span></span>

<span data-ttu-id="0aba8-107">В этой статье рассматривается вызов методов .NET из JavaScript.</span><span class="sxs-lookup"><span data-stu-id="0aba8-107">This article covers invoking .NET methods from JavaScript.</span></span> <span data-ttu-id="0aba8-108">Сведения о том, как вызывать функции JavaScript из .NET, см. в разделе <xref:blazor/call-javascript-from-dotnet>.</span><span class="sxs-lookup"><span data-stu-id="0aba8-108">For information on how to call JavaScript functions from .NET, see <xref:blazor/call-javascript-from-dotnet>.</span></span>

<span data-ttu-id="0aba8-109">[Просмотреть или скачать образец кода](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([как скачивать](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="0aba8-109">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

## <a name="static-net-method-call"></a><span data-ttu-id="0aba8-110">Вызов статического метода .NET</span><span class="sxs-lookup"><span data-stu-id="0aba8-110">Static .NET method call</span></span>

<span data-ttu-id="0aba8-111">Чтобы вызвать статический метод .NET из JavaScript, используйте функции `DotNet.invokeMethod` или `DotNet.invokeMethodAsync`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-111">To invoke a static .NET method from JavaScript, use the `DotNet.invokeMethod` or `DotNet.invokeMethodAsync` functions.</span></span> <span data-ttu-id="0aba8-112">Передайте идентификатор статического метода, который необходимо вызвать, имя сборки, содержащей функцию, и любые аргументы.</span><span class="sxs-lookup"><span data-stu-id="0aba8-112">Pass in the identifier of the static method you wish to call, the name of the assembly containing the function, and any arguments.</span></span> <span data-ttu-id="0aba8-113">Асинхронная версия является предпочтительной для поддержки сценариев Blazor Server.</span><span class="sxs-lookup"><span data-stu-id="0aba8-113">The asynchronous version is preferred to support Blazor Server scenarios.</span></span> <span data-ttu-id="0aba8-114">Метод .NET должен быть открытым, статическим и иметь атрибут [`[JSInvokable]`](xref:Microsoft.JSInterop.JSInvokableAttribute).</span><span class="sxs-lookup"><span data-stu-id="0aba8-114">The .NET method must be public, static, and have the [`[JSInvokable]`](xref:Microsoft.JSInterop.JSInvokableAttribute) attribute.</span></span> <span data-ttu-id="0aba8-115">Вызов открытых универсальных методов в настоящее время не поддерживается.</span><span class="sxs-lookup"><span data-stu-id="0aba8-115">Calling open generic methods isn't currently supported.</span></span>

<span data-ttu-id="0aba8-116">Пример приложения включает метод C#, возвращающий массив `int`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-116">The sample app includes a C# method to return an `int` array.</span></span> <span data-ttu-id="0aba8-117">К методу применяется атрибут [`[JSInvokable]`](xref:Microsoft.JSInterop.JSInvokableAttribute).</span><span class="sxs-lookup"><span data-stu-id="0aba8-117">The [`[JSInvokable]`](xref:Microsoft.JSInterop.JSInvokableAttribute) attribute is applied to the method.</span></span>

<span data-ttu-id="0aba8-118">`Pages/JsInterop.razor`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-118">`Pages/JsInterop.razor`:</span></span>

```razor
<button type="button" class="btn btn-primary"
        onclick="exampleJsFunctions.returnArrayAsyncJs()">
    Trigger .NET static method ReturnArrayAsync
</button>

@code {
    [JSInvokable]
    public static Task<int[]> ReturnArrayAsync()
    {
        return Task.FromResult(new int[] { 1, 2, 3 });
    }
}
```

<span data-ttu-id="0aba8-119">JavaScript, предоставляемый клиенту, вызывает метод C# .NET.</span><span class="sxs-lookup"><span data-stu-id="0aba8-119">JavaScript served to the client invokes the C# .NET method.</span></span>

<span data-ttu-id="0aba8-120">`wwwroot/exampleJsInterop.js`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-120">`wwwroot/exampleJsInterop.js`:</span></span>

[!code-javascript[](./common/samples/3.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=8-14)]

<span data-ttu-id="0aba8-121">Если нажата кнопка **`Trigger .NET static method ReturnArrayAsync`** , проверьте выходные данные консоли в средствах веб-разработчика браузера.</span><span class="sxs-lookup"><span data-stu-id="0aba8-121">When the **`Trigger .NET static method ReturnArrayAsync`** button is selected, examine the console output in the browser's web developer tools.</span></span>

<span data-ttu-id="0aba8-122">Выходные данные консоли:</span><span class="sxs-lookup"><span data-stu-id="0aba8-122">The console output is:</span></span>

```console
Array(4) [ 1, 2, 3, 4 ]
```

<span data-ttu-id="0aba8-123">Четвертое значение массива помещается в массив (`data.push(4);`), возвращаемый методом `ReturnArrayAsync`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-123">The fourth array value is pushed to the array (`data.push(4);`) returned by `ReturnArrayAsync`.</span></span>

<span data-ttu-id="0aba8-124">По умолчанию идентификатором метода является имя метода, но можно указать другой идентификатор с помощью атрибута конструктора [`[JSInvokable]`](xref:Microsoft.JSInterop.JSInvokableAttribute):</span><span class="sxs-lookup"><span data-stu-id="0aba8-124">By default, the method identifier is the method name, but you can specify a different identifier using the [`[JSInvokable]`](xref:Microsoft.JSInterop.JSInvokableAttribute) attribute constructor:</span></span>

```csharp
@code {
    [JSInvokable("DifferentMethodName")]
    public static Task<int[]> ReturnArrayAsync()
    {
        return Task.FromResult(new int[] { 1, 2, 3 });
    }
}
```

<span data-ttu-id="0aba8-125">В файле JavaScript на стороне клиента:</span><span class="sxs-lookup"><span data-stu-id="0aba8-125">In the client-side JavaScript file:</span></span>

```javascript
returnArrayAsyncJs: function () {
  DotNet.invokeMethodAsync('BlazorSample', 'DifferentMethodName')
    .then(data => {
      data.push(4);
      console.log(data);
    });
}
```

## <a name="instance-method-call"></a><span data-ttu-id="0aba8-126">Вызов метода экземпляра</span><span class="sxs-lookup"><span data-stu-id="0aba8-126">Instance method call</span></span>

<span data-ttu-id="0aba8-127">Кроме того, из JavaScript можно вызывать методы экземпляра .NET.</span><span class="sxs-lookup"><span data-stu-id="0aba8-127">You can also call .NET instance methods from JavaScript.</span></span> <span data-ttu-id="0aba8-128">Для этого необходимо выполнить следующие действия.</span><span class="sxs-lookup"><span data-stu-id="0aba8-128">To invoke a .NET instance method from JavaScript:</span></span>

* <span data-ttu-id="0aba8-129">Передайте в JavaScript экземпляр .NET по ссылке:</span><span class="sxs-lookup"><span data-stu-id="0aba8-129">Pass the .NET instance by reference to JavaScript:</span></span>
  * <span data-ttu-id="0aba8-130">Выполните статический вызов <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A?displayProperty=nameWithType>.</span><span class="sxs-lookup"><span data-stu-id="0aba8-130">Make a static call to <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A?displayProperty=nameWithType>.</span></span>
  * <span data-ttu-id="0aba8-131">Заключите экземпляр в экземпляр <xref:Microsoft.JSInterop.DotNetObjectReference> и вызовите метод <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A> в экземпляре <xref:Microsoft.JSInterop.DotNetObjectReference>.</span><span class="sxs-lookup"><span data-stu-id="0aba8-131">Wrap the instance in a <xref:Microsoft.JSInterop.DotNetObjectReference> instance and call <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A> on the <xref:Microsoft.JSInterop.DotNetObjectReference> instance.</span></span> <span data-ttu-id="0aba8-132">Удалите объекты <xref:Microsoft.JSInterop.DotNetObjectReference> (пример приведен далее в этом разделе).</span><span class="sxs-lookup"><span data-stu-id="0aba8-132">Dispose of <xref:Microsoft.JSInterop.DotNetObjectReference> objects (an example appears later in this section).</span></span>
* <span data-ttu-id="0aba8-133">Вызовите методы экземпляра .NET в экземпляре с помощью функций `invokeMethod` или `invokeMethodAsync`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-133">Invoke .NET instance methods on the instance using the `invokeMethod` or `invokeMethodAsync` functions.</span></span> <span data-ttu-id="0aba8-134">Экземпляр .NET можно также передать в качестве аргумента при вызове других методов .NET из JavaScript.</span><span class="sxs-lookup"><span data-stu-id="0aba8-134">The .NET instance can also be passed as an argument when invoking other .NET methods from JavaScript.</span></span>

> [!NOTE]
> <span data-ttu-id="0aba8-135">Пример приложения записывает сообщения в консоль на стороне клиента.</span><span class="sxs-lookup"><span data-stu-id="0aba8-135">The sample app logs messages to the client-side console.</span></span> <span data-ttu-id="0aba8-136">В следующих примерах, продемонстрированных в учебном приложении, изучите выходные данные консоли браузера в средствах разработчика браузера.</span><span class="sxs-lookup"><span data-stu-id="0aba8-136">For the following examples demonstrated by the sample app, examine the browser's console output in the browser's developer tools.</span></span>

<span data-ttu-id="0aba8-137">Если нажата кнопка **`Trigger .NET instance method HelloHelper.SayHello`** , вызывается `ExampleJsInterop.CallHelloHelperSayHello` и в метод передается имя `Blazor`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-137">When the **`Trigger .NET instance method HelloHelper.SayHello`** button is selected, `ExampleJsInterop.CallHelloHelperSayHello` is called and passes a name, `Blazor`, to the method.</span></span>

<span data-ttu-id="0aba8-138">`Pages/JsInterop.razor`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-138">`Pages/JsInterop.razor`:</span></span>

```razor
<button type="button" class="btn btn-primary" @onclick="TriggerNetInstanceMethod">
    Trigger .NET instance method HelloHelper.SayHello
</button>

@code {
    public async Task TriggerNetInstanceMethod()
    {
        var exampleJsInterop = new ExampleJsInterop(JSRuntime);
        await exampleJsInterop.CallHelloHelperSayHello("Blazor");
    }
}
```

<span data-ttu-id="0aba8-139">Метод `CallHelloHelperSayHello` вызывает функцию JavaScript `sayHello` с новым экземпляром `HelloHelper`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-139">`CallHelloHelperSayHello` invokes the JavaScript function `sayHello` with a new instance of `HelloHelper`.</span></span>

<span data-ttu-id="0aba8-140">`JsInteropClasses/ExampleJsInterop.cs`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-140">`JsInteropClasses/ExampleJsInterop.cs`:</span></span>

[!code-csharp[](./common/samples/3.x/BlazorWebAssemblySample/JsInteropClasses/ExampleJsInterop.cs?name=snippet1&highlight=11-18)]

<span data-ttu-id="0aba8-141">`wwwroot/exampleJsInterop.js`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-141">`wwwroot/exampleJsInterop.js`:</span></span>

[!code-javascript[](./common/samples/3.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=15-18)]

<span data-ttu-id="0aba8-142">Имя передается в конструктор `HelloHelper`, который задает свойство `HelloHelper.Name`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-142">The name is passed to `HelloHelper`'s constructor, which sets the `HelloHelper.Name` property.</span></span> <span data-ttu-id="0aba8-143">При выполнении функции JavaScript `sayHello` метод `HelloHelper.SayHello` возвращает сообщение `Hello, {Name}!`, которое записывается в консоль функцией JavaScript.</span><span class="sxs-lookup"><span data-stu-id="0aba8-143">When the JavaScript function `sayHello` is executed, `HelloHelper.SayHello` returns the `Hello, {Name}!` message, which is written to the console by the JavaScript function.</span></span>

<span data-ttu-id="0aba8-144">`JsInteropClasses/HelloHelper.cs`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-144">`JsInteropClasses/HelloHelper.cs`:</span></span>

[!code-csharp[](./common/samples/3.x/BlazorWebAssemblySample/JsInteropClasses/HelloHelper.cs?name=snippet1&highlight=5,10-11)]

<span data-ttu-id="0aba8-145">Выходные данные консоли в средствах веб-разработчика браузера:</span><span class="sxs-lookup"><span data-stu-id="0aba8-145">Console output in the browser's web developer tools:</span></span>

```console
Hello, Blazor!
```

<span data-ttu-id="0aba8-146">Чтобы избежать утечки памяти и разрешить сборку мусора для компонента, который создает метод <xref:Microsoft.JSInterop.DotNetObjectReference>, используйте один из следующих подходов.</span><span class="sxs-lookup"><span data-stu-id="0aba8-146">To avoid a memory leak and allow garbage collection on a component that creates a <xref:Microsoft.JSInterop.DotNetObjectReference>, adopt one of the following approaches:</span></span>

* <span data-ttu-id="0aba8-147">Удалите объект в классе, который создал экземпляр <xref:Microsoft.JSInterop.DotNetObjectReference>.</span><span class="sxs-lookup"><span data-stu-id="0aba8-147">Dispose of the object in the class that created the <xref:Microsoft.JSInterop.DotNetObjectReference> instance:</span></span>

  ```csharp
  public class ExampleJsInterop : IDisposable
  {
      private readonly IJSRuntime jsRuntime;
      private DotNetObjectReference<HelloHelper> objRef;

      public ExampleJsInterop(IJSRuntime jsRuntime)
      {
          this.jsRuntime = jsRuntime;
      }

      public ValueTask<string> CallHelloHelperSayHello(string name)
      {
          objRef = DotNetObjectReference.Create(new HelloHelper(name));

          return jsRuntime.InvokeAsync<string>(
              "exampleJsFunctions.sayHello",
              objRef);
      }

      public void Dispose()
      {
          objRef?.Dispose();
      }
  }
  ```

  <span data-ttu-id="0aba8-148">Предыдущий шаблон, показанный в классе `ExampleJsInterop`, также можно реализовать в компоненте:</span><span class="sxs-lookup"><span data-stu-id="0aba8-148">The preceding pattern shown in the `ExampleJsInterop` class can also be implemented in a component:</span></span>

  ```razor
  @page "/JSInteropComponent"
  @using BlazorSample.JsInteropClasses
  @implements IDisposable
  @inject IJSRuntime JSRuntime

  <h1>JavaScript Interop</h1>

  <button type="button" class="btn btn-primary" @onclick="TriggerNetInstanceMethod">
      Trigger .NET instance method HelloHelper.SayHello
  </button>

  @code {
      private DotNetObjectReference<HelloHelper> objRef;

      public async Task TriggerNetInstanceMethod()
      {
          objRef = DotNetObjectReference.Create(new HelloHelper("Blazor"));

          await JSRuntime.InvokeAsync<string>(
              "exampleJsFunctions.sayHello",
              objRef);
      }

      public void Dispose()
      {
          objRef?.Dispose();
      }
  }
  ```

* <span data-ttu-id="0aba8-149">Если компонент или класс не удаляет <xref:Microsoft.JSInterop.DotNetObjectReference>, удалите объект в клиенте, вызвав `.dispose()`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-149">When the component or class doesn't dispose of the <xref:Microsoft.JSInterop.DotNetObjectReference>, dispose of the object on the client by calling `.dispose()`:</span></span>

  ```javascript
  window.myFunction = (dotnetHelper) => {
    dotnetHelper.invokeMethod('BlazorSample', 'MyMethod');
    dotnetHelper.dispose();
  }
  ```

## <a name="component-instance-method-call"></a><span data-ttu-id="0aba8-150">Вызов метода экземпляра компонента</span><span class="sxs-lookup"><span data-stu-id="0aba8-150">Component instance method call</span></span>

<span data-ttu-id="0aba8-151">Порядок вызова методов .NET компонента</span><span class="sxs-lookup"><span data-stu-id="0aba8-151">To invoke a component's .NET methods:</span></span>

* <span data-ttu-id="0aba8-152">Используйте функцию `invokeMethod` или `invokeMethodAsync`, чтобы вызвать статический метод для компонента.</span><span class="sxs-lookup"><span data-stu-id="0aba8-152">Use the `invokeMethod` or `invokeMethodAsync` function to make a static method call to the component.</span></span>
* <span data-ttu-id="0aba8-153">Статический метод компонента создает программу-оболочку вызова метода экземпляра в виде вызванного <xref:System.Action>.</span><span class="sxs-lookup"><span data-stu-id="0aba8-153">The component's static method wraps the call to its instance method as an invoked <xref:System.Action>.</span></span>

<span data-ttu-id="0aba8-154">В JavaScript на стороне клиента:</span><span class="sxs-lookup"><span data-stu-id="0aba8-154">In the client-side JavaScript:</span></span>

```javascript
function updateMessageCallerJS() {
  DotNet.invokeMethod('BlazorSample', 'UpdateMessageCaller');
}
```

<span data-ttu-id="0aba8-155">`Pages/JSInteropComponent.razor`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-155">`Pages/JSInteropComponent.razor`:</span></span>

```razor
@page "/JSInteropComponent"

<p>
    Message: @message
</p>

<p>
    <button onclick="updateMessageCallerJS()">Call JS Method</button>
</p>

@code {
    private static Action action;
    private string message = "Select the button.";

    protected override void OnInitialized()
    {
        action = UpdateMessage;
    }

    private void UpdateMessage()
    {
        message = "UpdateMessage Called!";
        StateHasChanged();
    }

    [JSInvokable]
    public static void UpdateMessageCaller()
    {
        action.Invoke();
    }
}
```

<span data-ttu-id="0aba8-156">При наличии нескольких компонентов, каждый из которых содержит методы экземпляра, используйте вспомогательный класс для вызова методов экземпляра (как <xref:System.Action>) для каждого компонента.</span><span class="sxs-lookup"><span data-stu-id="0aba8-156">When there are several components, each with instance methods to call, use a helper class to invoke the instance methods (as <xref:System.Action>s) of each component.</span></span>

<span data-ttu-id="0aba8-157">В следующем примере:</span><span class="sxs-lookup"><span data-stu-id="0aba8-157">In the following example:</span></span>

* <span data-ttu-id="0aba8-158">Компонент `JSInteropExample` содержит несколько компонентов `ListItem`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-158">The `JSInteropExample` component contains several `ListItem` components.</span></span>
* <span data-ttu-id="0aba8-159">Каждый компонент `ListItem` состоит из сообщения и кнопки.</span><span class="sxs-lookup"><span data-stu-id="0aba8-159">Each `ListItem` component is composed of a message and a button.</span></span>
* <span data-ttu-id="0aba8-160">При выборе кнопки компонента `ListItem` метод `UpdateMessage` `ListItem`изменяет текст элемента списка и скрывает кнопку.</span><span class="sxs-lookup"><span data-stu-id="0aba8-160">When a `ListItem` component button is selected, that `ListItem`'s `UpdateMessage` method changes the list item text and hides the button.</span></span>

<span data-ttu-id="0aba8-161">`MessageUpdateInvokeHelper.cs`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-161">`MessageUpdateInvokeHelper.cs`:</span></span>

```csharp
using System;
using Microsoft.JSInterop;

public class MessageUpdateInvokeHelper
{
    private Action action;

    public MessageUpdateInvokeHelper(Action action)
    {
        action = action;
    }

    [JSInvokable("BlazorSample")]
    public void UpdateMessageCaller()
    {
        action.Invoke();
    }
}
```

<span data-ttu-id="0aba8-162">В JavaScript на стороне клиента:</span><span class="sxs-lookup"><span data-stu-id="0aba8-162">In the client-side JavaScript:</span></span>

```javascript
window.updateMessageCallerJS = (dotnetHelper) => {
    dotnetHelper.invokeMethod('BlazorSample', 'UpdateMessageCaller');
    dotnetHelper.dispose();
}
```

<span data-ttu-id="0aba8-163">`Shared/ListItem.razor`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-163">`Shared/ListItem.razor`:</span></span>

```razor
@inject IJSRuntime JsRuntime

<li>
    @message
    <button @onclick="InteropCall" style="display:@display">InteropCall</button>
</li>

@code {
    private string message = "Select one of these list item buttons.";
    private string display = "inline-block";
    private MessageUpdateInvokeHelper messageUpdateInvokeHelper;

    protected override void OnInitialized()
    {
        messageUpdateInvokeHelper = new MessageUpdateInvokeHelper(UpdateMessage);
    }

    protected async Task InteropCall()
    {
        await JsRuntime.InvokeVoidAsync("updateMessageCallerJS",
            DotNetObjectReference.Create(messageUpdateInvokeHelper));
    }

    private void UpdateMessage()
    {
        message = "UpdateMessage Called!";
        display = "none";
        StateHasChanged();
    }
}
```

<span data-ttu-id="0aba8-164">`Pages/JSInteropExample.razor`.</span><span class="sxs-lookup"><span data-stu-id="0aba8-164">`Pages/JSInteropExample.razor`:</span></span>

```razor
@page "/JSInteropExample"

<h1>List of components</h1>

<ul>
    <ListItem />
    <ListItem />
    <ListItem />
    <ListItem />
</ul>
```

[!INCLUDE[Share interop code in a class library](~/includes/blazor-share-interop-code.md)]

## <a name="avoid-circular-object-references"></a><span data-ttu-id="0aba8-165">Исключение циклических ссылок на объекты</span><span class="sxs-lookup"><span data-stu-id="0aba8-165">Avoid circular object references</span></span>

<span data-ttu-id="0aba8-166">Объекты, содержащие циклические ссылки, не могут быть сериализованы на клиенте для:</span><span class="sxs-lookup"><span data-stu-id="0aba8-166">Objects that contain circular references can't be serialized on the client for either:</span></span>

* <span data-ttu-id="0aba8-167">вызовов метода .NET.</span><span class="sxs-lookup"><span data-stu-id="0aba8-167">.NET method calls.</span></span>
* <span data-ttu-id="0aba8-168">Вызов метода JavaScript из C#, если тип возвращаемого значения имеет циклические ссылки.</span><span class="sxs-lookup"><span data-stu-id="0aba8-168">JavaScript method calls from C# when the return type has circular references.</span></span>

<span data-ttu-id="0aba8-169">Дополнительные сведения см. на следующих страницах:</span><span class="sxs-lookup"><span data-stu-id="0aba8-169">For more information, see the following issues:</span></span>

* [<span data-ttu-id="0aba8-170">Циклические ссылки не поддерживаются, вторая серия (DotNet/aspnetcore #20525)</span><span class="sxs-lookup"><span data-stu-id="0aba8-170">Circular references are not supported, take two (dotnet/aspnetcore #20525)</span></span>](https://github.com/dotnet/aspnetcore/issues/20525)
* [<span data-ttu-id="0aba8-171">Предложение. Добавить механизм для обработки циклических ссылок при сериализации (dotnet/runtime #30820)</span><span class="sxs-lookup"><span data-stu-id="0aba8-171">Proposal: Add mechanism to handle circular references when serializing (dotnet/runtime #30820)</span></span>](https://github.com/dotnet/runtime/issues/30820)

## <a name="additional-resources"></a><span data-ttu-id="0aba8-172">Дополнительные ресурсы</span><span class="sxs-lookup"><span data-stu-id="0aba8-172">Additional resources</span></span>

* <xref:blazor/call-javascript-from-dotnet>
* [<span data-ttu-id="0aba8-173">Пример `InteropComponent.razor` (репозиторий GitHub dotnet/AspNetCore, ветвь выпуска 3.1)</span><span class="sxs-lookup"><span data-stu-id="0aba8-173">`InteropComponent.razor` example (dotnet/AspNetCore GitHub repository, 3.1 release branch)</span></span>](https://github.com/dotnet/AspNetCore/blob/release/3.1/src/Components/test/testassets/BasicTestApp/InteropComponent.razor)
* <span data-ttu-id="0aba8-174">[Выполнение передачи больших объемов данных в приложениях Blazor Server](xref:blazor/advanced-scenarios#perform-large-data-transfers-in-blazor-server-apps)</span><span class="sxs-lookup"><span data-stu-id="0aba8-174">[Perform large data transfers in Blazor Server apps](xref:blazor/advanced-scenarios#perform-large-data-transfers-in-blazor-server-apps)</span></span>
