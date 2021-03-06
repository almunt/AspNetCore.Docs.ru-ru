Шаблон по умолчанию создает ссылки и страницы **RazorPagesMovie**, **Главная**, **О программе** и **Контакт**. В зависимости от размера окна браузера может потребоваться щелкнуть значок навигации, чтобы отобразить ссылки.

![Домашняя или индексная страница](../../tutorials/razor-pages/razor-pages-start/_static/home2.png)

Проверьте ссылки. Ссылки **RazorPagesMovie** и **Главная** ведут на страницу индексов. Ссылки **О программе** и **Контакт** указывают соответственно на страницы `About` и `Contact`.

## <a name="project-files-and-folders"></a>Защита файлов и папок

В следующей таблице перечислены файлы и папки в проекте. В рамках этого учебника важнее всего разобраться с файлом *Startup.cs*. Каждую из указанных ниже ссылок просматривать не требуется. Ссылки приводятся для справки и позволяют получить дополнительную информацию о каком-либо файле или папке в проекте.

| Файл или папка              | Цель |
| ----------------- | ------------ |
| wwwroot | Содержит статические файлы. См. [Статические файлы](xref:fundamentals/static-files). |
| Pages | Папка для [Razor Pages](xref:razor-pages/index). |
| *appsettings.json* | [Конфигурация](xref:fundamentals/configuration/index) |
| *Program.cs* | [Содержит](xref:fundamentals/host/index) приложение ASP.NET Core.|
| *Startup.cs* | Настраивает службы и конвейер обработки запросов. См. раздел [Запуск](xref:fundamentals/startup).|

### <a name="the-pages-folder"></a>Папка "Pages"

Файл *_Layout.cshtml* содержит общие элементы HTML (скрипты и таблицу стилей) и определяет макет для приложения. Например, при выборе ссылка **RazorPagesMovie**, **Главная**, **О программе** или **Контакты** отображаются одинаковые элементы. В число общих элементов входят меню навигации наверху экрана и заголовок внизу окна. Дополнительные сведения см. в статье о [макете](xref:mvc/views/layout).

Файл *_ViewImports.cshtml* содержит директивы Razor, импортированные в каждую страницу Razor Pages. Дополнительные сведения см. в разделе [Импорт общих директив](xref:mvc/views/layout#importing-shared-directives).

Файл *_ViewStart.cshtml* определяет свойство Razor Pages `Layout`, необходимое для использования файла *_Layout.cshtml*. Дополнительные сведения см. в статье о [макете](xref:mvc/views/layout).

Файл *_ValidationScriptsPartial.cshtml* содержит ссылку на сценарии проверки [jQuery](https://jquery.com/). Когда далее в этом учебнике мы добавим страницы `Create` и `Edit`, будет использоваться файл *_ValidationScriptsPartial.cshtml*.

`About`, `Contact` и `Index` — это базовые страницы, которые можно использовать для запуска приложения. Страница `Error` используется для отображения сведений об ошибках. Страница `Privacy` позволяет указать сведения о политике конфиденциальности веб-сайта.
