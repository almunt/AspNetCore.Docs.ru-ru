---
title: DevOps с помощью ASP.NET Core и Azure | Мониторинг и отладка
author: CamSoper
description: Рекомендации по созданию сквозного решения конвейера DevOps для приложения ASP.NET Core, размещенного в Azure.
ms.author: casoper
ms.date: 08/07/2018
uid: azure/devops/monitor
ms.openlocfilehash: c2fc88493aee04d7ea2781d17e808581e89d2082
ms.sourcegitcommit: 29dfe436f54a27fbb4f6494bc639d16c75001fab
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/09/2018
ms.locfileid: "42909606"
---
# <a name="monitor-and-debug"></a>Мониторинг и отладка

Произведя развертывание приложения и построение конвейера DevOps, важно понять, как выполнять мониторинг и устранение неполадок приложения.

В этом разделе вы выполните следующие задачи:

* Найти базовый мониторинг и устранение неполадок данных на портале Azure
* Узнайте, как Azure Monitor обеспечивает более подробное рассмотрение метрики во всех службах Azure
* Подключение веб-приложения с помощью Application Insights для профилирования приложения
* Включите ведение журнала и узнать, где можно скачать журналы
* Stream журналами в реальном времени
* Узнать, где можно настроить оповещения
* Дополнительные сведения об удаленной отладки веб-приложений службы приложений Azure.

## <a name="basic-monitoring-and-troubleshooting"></a>Базовый мониторинг и устранение неполадок

Веб-приложениях службы приложений, легко отслеживаются в реальном времени. Портал Azure отображает метрики в простой для понимания диаграмм и графиков.

1. Откройте [портала Azure](https://portal.azure.com), а затем перейдите к *mywebapp\<unique_number\>*  службы приложений.

1. **Обзор** вкладка отображает полезные сведения «в краткие», включая диаграммы, отображения последних метрик.

    ![Панель "Обзор"](./media/monitoring/overview.png)

    * **HTTP 5xx**: число ошибок на стороне сервера, обычно исключения в коде ASP.NET Core.
    * **Данные в**: объем входящих данных, поступающих на веб-приложения.
    * **Исходящие данные**: данных исходящий трафик от веб-приложения на клиентах.
    * **Запросы**: число HTTP-запросов.
    * **Среднее время ответа**: среднее время для веб-приложения отвечать на запросы HTTP.

    Несколько средств самообслуживания, для устранения неполадок и оптимизации, можно также найти на этой странице.

    ![Средства самообслуживания](./media/monitoring/wizards.png)

    * **Диагностика и устранение неполадок** — это средство самостоятельного устранения неполадок.
    * **Application Insights** — для профилирования производительности и поведение приложения и рассматривается далее в этом разделе.
    * **Помощник по службе приложений** делает рекомендации для настройки работы приложения.

## <a name="advanced-monitoring"></a>Расширенный мониторинг

[Azure Monitor](https://docs.microsoft.com/azure/monitoring-and-diagnostics/) — централизованная служба для всех метрик мониторинга и настройки оповещений между службами Azure. В Azure Monitor администраторы могут детально отслеживать производительность и определять тенденции. Каждая служба Azure предлагает собственную [набор метрик](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-supported-metrics#microsoftwebsites-excluding-functions) в Azure Monitor.

## <a name="profile-with-application-insights"></a>Профиль с помощью Application Insights

[Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-overview) — это служба Azure для анализа производительности и стабильности веб-приложений и как их использовать пользователи. Данные из Application Insights — исследование, отличный от Azure Monitor. Данные можно предоставить, разработчики и Администраторы получают необходимую информацию для улучшения приложений. Можно добавить Application Insights для ресурса службы приложений Azure без изменения кода.

1. Откройте [портала Azure](https://portal.azure.com), а затем перейдите к *mywebapp\<unique_number\>*  службы приложений.
1. Из **Обзор** щелкните **Application Insights** плитку.

    ![Плитку службы Application Insights](./media/monitoring/app-insights.png)

1. Выберите **создать новый ресурс** "переключатель". Используемое имя ресурса по умолчанию и выберите расположение для ресурса Application Insights. Расположение не обязательно должен соответствовать, веб-приложения.

    ![Программа установки Application Insights](./media/monitoring/new-app-insights.png)

1. Для **среда выполнения или платформа**выберите **ASP.NET Core**. Примите параметры по умолчанию.
1. Нажмите кнопку **ОК**. Если запрос на подтверждение, установите **Продолжить**.
1. После создания ресурса, щелкните имя ресурса Application Insights, чтобы перейти непосредственно к странице Application Insights.

    ![Новый ресурс Application Insights будет готов](./media/monitoring/new-app-insights-done.png)

Так как используется приложение, данные накапливаются. Выберите **обновить** перезагрузить в колонке с новыми данными.

![Вкладка обзора Application Insights](./media/monitoring/app-insights-overview.png)

Application Insights предоставляет полезную информацию на сервере без дополнительной настройки. Чтобы получить максимальную пользу из Application Insights, [инструментирование приложения с помощью пакета SDK Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net-core). При правильной настройке эта служба предоставляет end-to-end monitoring в веб-сервера и браузера, включая производительности на стороне клиента. Дополнительные сведения см. в разделе [документация по Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-overview).

## <a name="logging"></a>Ведение журнала

Веб-журналы сервера и приложения будут отключены по умолчанию в службе приложений Azure. Включите журналы, выполнив следующие действия.

1. Откройте [портала Azure](https://portal.azure.com)и перейдите к *mywebapp\<unique_number\>*  службы приложений.
1. В меню слева, прокрутите вниз до раздела **мониторинг** раздел. Выберите **журналы диагностики**.

    ![Ссылка на журналы диагностики](./media/monitoring/logging.png)

1. Включите **ведение журнала приложения (файловая система)**. При появлении соответствующего запроса, щелкните окно, чтобы установить расширения, чтобы позволить приложению, ведение журнала в веб-приложения.
1. Задайте **журналы веб-сервера** для **файловой системы**.
1. Введите **срок** в днях. Например, 30.
1. Нажмите кнопку **Сохранить**.

ASP.NET Core и веб-сервер (служба приложений) журналы создаются для веб-приложения. Их можно загрузить с помощью FTP или FTPS информация. Пароль совпадает со значением учетные данные развертывания, созданную ранее в этом руководстве. Журналы могут быть [потоковом режиме непосредственно на локальном компьютере с помощью PowerShell или интерфейса командной строки Azure](https://docs.microsoft.com/azure/app-service/web-sites-enable-diagnostic-log#download). Журналы также может быть [просмотреть в Application Insights](https://docs.microsoft.com/azure/app-service/web-sites-enable-diagnostic-log#how-to-view-logs-in-application-insights).

## <a name="log-streaming"></a>Потоковая передача журналов

Приложения и веб-журналы сервера может передаваться в режиме реального времени на портале.

1. Откройте [портала Azure](https://portal.azure.com)и перейдите к *mywebapp\<unique_number\>*  службы приложений.
1. В меню слева, прокрутите вниз до раздела **мониторинг** и выберите **поток журналов**.

    ![Ссылка журнала потока](./media/monitoring/log-stream.png)

Журналы также может быть [потоковую передачу с помощью Azure CLI или Azure PowerShell](https://docs.microsoft.com/azure/app-service/web-sites-enable-diagnostic-log#streamlogs), в том числе через Cloud Shell.

## <a name="alerts"></a>Предупреждения

Azure Monitor также предоставляет [оповещения в режиме реального времени](https://docs.microsoft.com/azure/monitoring-and-diagnostics/insights-alerts-portal) на основе метрик, административные события и другим критериям.

> *Примечание: В настоящее время оповещения о показателей веб-приложения доступен только в службе оповещений (Классическая модель).*

[Оповещений (Классическая) служба](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitor-quick-resource-metric-alert-portal) можно найти в Azure Monitor или в разделе **мониторинг** раздел параметров приложения службы.

![Ссылка на уведомления (классический)](./media/monitoring/alerts.png)

## <a name="live-debugging"></a>Динамическая Отладка

Служба приложений Azure может быть [отлаживать удаленно с помощью Visual Studio](https://docs.microsoft.com/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio#remotedebug) когда журналы не предоставляли достаточно информации. Тем не менее для удаленной отладки требуется приложение компилируется с отладочными символами. Отладка не должно выполняться в рабочей среде, за исключением в качестве последнего средства.

## <a name="conclusion"></a>Заключение

В этом разделе вы выполнили следующие задачи:

* Найти базовый мониторинг и устранение неполадок данных на портале Azure
* Узнайте, как Azure Monitor обеспечивает более подробное рассмотрение метрики во всех службах Azure
* Подключение веб-приложения с помощью Application Insights для профилирования приложения
* Включите ведение журнала и узнать, где можно скачать журналы
* Stream журналами в реальном времени
* Узнать, где можно настроить оповещения
* Дополнительные сведения об удаленной отладки веб-приложений службы приложений Azure.

## <a name="additional-reading"></a>Дополнительные материалы для чтения

* [Устранение неполадок ASP.NET Core в службе приложений Azure](https://docs.microsoft.com/aspnet/core/host-and-deploy/azure-apps/troubleshoot)
* [Справочник по общим ошибкам для службы приложений Azure и служб IIS с ASP.NET Core](https://docs.microsoft.com/aspnet/core/host-and-deploy/azure-iis-errors-reference)
* [Мониторинг производительности Azure веб-приложения с помощью Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-azure-web-apps)
* [Включение функции ведения журналов диагностики для веб-приложений в службе приложений Azure](https://docs.microsoft.com/azure/app-service/web-sites-enable-diagnostic-log)
* [Устранение неполадок веб-приложения в службе приложений Azure с помощью Visual Studio](https://docs.microsoft.com/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio)
* [Создание классических оповещений метрик в Azure Monitor для служб Azure — портал Azure](https://docs.microsoft.com/azure/monitoring-and-diagnostics/insights-alerts-portal)