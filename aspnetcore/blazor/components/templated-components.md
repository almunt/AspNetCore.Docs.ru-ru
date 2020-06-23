---
title: Шаблонные компоненты Blazor в ASP.NET Core
author: guardrex
description: Сведения о шаблонных компонентах, которые могут принимать один или несколько шаблонов пользовательского интерфейса в качестве параметров, используемых затем как часть логики отрисовки компонента.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/18/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/components/templated-components
ms.openlocfilehash: ae591fb8280b706d568dd530e2e60a2f7955841c
ms.sourcegitcommit: 490434a700ba8c5ed24d849bd99d8489858538e3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/19/2020
ms.locfileid: "85103448"
---
# <a name="aspnet-core-blazor-templated-components"></a>Шаблонные компоненты Blazor в ASP.NET Core

Авторы: [Люк Латэм](https://github.com/guardrex) (Luke Latham) и [Дэниэл Рот](https://github.com/danroth27) (Daniel Roth)

Шаблонные компоненты — это компоненты, которые могут принимать один или несколько шаблонов пользовательского интерфейса в качестве параметров, используемых затем как часть логики отрисовки компонента. Шаблонные компоненты позволяют разрабатывать более общие компоненты, которые удобнее использовать повторно, чем обычные. Вот несколько примеров:

* компонент таблицы, позволяющий пользователю задавать шаблоны для заголовка, строк и нижнего колонтитула таблицы;
* компонент списка, позволяющий пользователю задавать шаблон для отрисовываемых элементов списка.

## <a name="template-parameters"></a>Параметры шаблона

Шаблонный компонент определяется путем указания одного или нескольких параметров компонента типа <xref:Microsoft.AspNetCore.Components.RenderFragment> или <xref:Microsoft.AspNetCore.Components.RenderFragment%601>. Фрагмент отрисовки представляет часть пользовательского интерфейса, подлежащую отрисовке. <xref:Microsoft.AspNetCore.Components.RenderFragment%601> принимает параметр типа, который можно задать при вызове фрагмента отрисовки.

Компонент `TableTemplate`:

[!code-razor[](../common/samples/3.x/BlazorWebAssemblySample/Components/TableTemplate.razor)]

При использовании шаблонного компонента параметры шаблона можно задать с помощью дочерних элементов, имена которых совпадают с именами параметров (`TableHeader` и `RowTemplate` в следующем примере):

```razor
<TableTemplate Items="pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@context.PetId</td>
        <td>@context.Name</td>
    </RowTemplate>
</TableTemplate>
```

> [!NOTE]
> Ограничения универсального типа будут поддерживаться в будущих выпусках. Дополнительные сведения см. в разделе [Разрешение ограничений универсального типа (dotnet/aspnetcore #8433)](https://github.com/dotnet/aspnetcore/issues/8433).

## <a name="template-context-parameters"></a>Параметры контекста шаблона

Аргументы компонента типа <xref:Microsoft.AspNetCore.Components.RenderFragment%601>, передаваемые как элементы, имеют неявный параметр `context` (например, `@context.PetId` в предыдущем примере кода), но его имя можно изменить с помощью атрибута `Context` дочернего элемента. В следующем примере атрибут `Context` элемента `RowTemplate` задает имя параметра `pet`:

```razor
<TableTemplate Items="pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate Context="pet">
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>
```

Кроме того, атрибут `Context` можно задать для элемента компонента. Заданный атрибут `Context` применяется ко всем указанным параметрам шаблона. Это может быть полезно, если необходимо указать имя параметра содержимого для неявного дочернего содержимого (без содержащего дочернего элемента). В следующем примере элемент `TableTemplate` имеет атрибут `Context`, который применяется ко всем параметрам шаблона:

```razor
<TableTemplate Items="pets" Context="pet">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>
```

## <a name="generic-typed-components"></a>Компоненты универсального типа

Шаблонные компоненты часто имеют универсальный тип. Например, универсальный компонент `ListViewTemplate` можно использовать для отрисовки значений `IEnumerable<T>`. Чтобы определить универсальный компонент, используйте директиву [`@typeparam`](xref:mvc/views/razor#typeparam) и укажите параметры типа:

[!code-razor[](../common/samples/3.x/BlazorWebAssemblySample/Components/ListViewTemplate.razor)]

При использовании компонентов универсального типа параметр типа выводится, если это возможно:

```razor
<ListViewTemplate Items="pets">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```

В противном случае параметр типа необходимо указать явно с помощью атрибута, совпадающего с именем параметра типа. В следующем примере `TItem="Pet"` задает тип:

```razor
<ListViewTemplate Items="pets" TItem="Pet">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```
