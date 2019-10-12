Выполните следующие команды интерфейса командной строки .NET Core:

```dotnetcli
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.SQLite
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

Приведенные выше команды добавляют следующие компоненты:

* [средство формирования шаблонов](xref:fundamentals/tools/dotnet-aspnet-codegenerator);
* средства Entity Framework Core для .NET Core CLI;
* поставщик EF Core для SQLite, который устанавливает пакет EF Core в качестве зависимости;
* Пакеты, необходимые для формирования шаблонов: `Microsoft.VisualStudio.Web.CodeGeneration.Design` и `Microsoft.EntityFrameworkCore.SqlServer`.