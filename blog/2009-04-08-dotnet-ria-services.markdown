---
title: .NET RIA Services
tags: [asp.net, ria services, silverlight]
src: http://cognitex.blogspot.ru/2009/04/net-ria-services.html
---
# .NET RIA Services
RIA - Rich Internet Application, а [.NET RIA Services] (http://www.microsoft.com/downloads/details.aspx?FamilyID=76bb3a07-3846-4564-b0c3-27972bcaabce&displaylang=en) - это дополнение к существующему DAL (Data Access Layer) в ASP.NET-приложениях. Основное назначение - объединение Silverlight и ASP.NET приложений.

Ниже пример организации обмена данными между Silverlight- и ASP.NET-приложениями:
<ol>
  <li>В Visual Studio 2008: Ctrl+Shift+N - Project Types: Silverlight - Template: Silverlight Application - OK</li>
  <li>В Solution Explorer выделить SilverlightApplication1.Web - Ctrl+Shift+A - Categories: Web - Templates: Domain Service Class - Add; в проект будет добавлен DomainService1.cs</li>
  <li>В класс DomainService1 добавить:</li>
</ol>
```c#
[ServiceOperation] public string Test(string value) { return "Hello " + value; }
```
<ol start="4">
  <li>Shift+F6 -- компиляция проекта</li>
  <li>В Solution Explorer выделить SilverlightApplication1 и нажать на Show All Files (вторая кнопка слева в toolbar'е) -- в проекте отобразится Generated_Code с файлом SilverlightApplication1.Web.g.cs</li>
  <li>Открыть файл MainPage.xaml.cs и в конструктор класса MainPage добавить:</li>
</ol>
```c#
SilverlightApplication1.Web.DomainService1 ds = new SilverlightApplication1.Web.DomainService1();
ds.TestCompleted += (s, e) => System.Windows.Browser.HtmlPage.Window.Alert("" + e.ReturnValue);
ds.Test("123");
```
<ol start="7">
  <li>F5 -- в результате в IE появится "Hello 123", полученное из метода DomainService1.Test</li>
</ol>
Для разработчиков .NET RIA Services открыт [форум (en)] (http://silverlight.net/forums/53.aspx).

P.S.
В Web.config в httpHandlers должен быть указан тег:
```xml
<add path="DataService.axd" verb="GET,POST" type="System.Web.Ria.DataServiceFactory, System.Web.Ria, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" validate="false"/>
```
Иначе получим исключение: System.Windows.Ria.Data.EntityOperationException: The specified resource was not found.
