# Подключение сервиса к работающему сайту
Предположим, что есть ASP.NET сайт, созданный в Visual Studio 2008: 
<ol>
  <li>Ctrl+Shift+N</li>
  <li>Project Types: Web, Templates: ASP.NET Web Application</li>
  <li>F5 - в результате откроется IE, в котором отобразится default.aspx</li>
</ol>
Например, необходимо к работающему сайту подключить сервис с методом GetTicks, который доступен по адресу /DataProvider.svc/Now и возращает значение DateTime.Now.Ticks в xml-формате.

Чтобы создать сервис в Visual Studio надо: 
<ol>
  <li>создать новый проект Project Types: Windows, Templates: Class Library</li>
  <li>в References добавить сборки: System.ServiceModel и System.ServiceModel.Web</li>
  <li>в проект добавить следующий код:</li>
</ol>
```c#
using System;
using System.ServiceModel;
using System.ServiceModel.Web;

namespace Providers
{
    	[ServiceContract]
    	public class DataProvider
    	{
        	[OperationContract]
        	[WebGet(UriTemplate = "/Now/", ResponseFormat = WebMessageFormat.Xml)]
        	public long GetTicks()
        	{
            	return DateTime.Now.Ticks;
        	}
    	}
}
```
<ol start="4">
  <li>Shift+F6 - в результате проект будет откомпилирован и в папке \Debug\bin появится файл Service.dll</li>
</ol>
Чтобы подключить сервис к сайту надо: 
<ol>
  <li>в папке с сайтом создать текстовый файл DataProvider.svc со следующим кодом:</li>
</ol>
```aspx
<%@ ServiceHost Service="Providers.DataProvider" Factory="System.ServiceModel.Activation.WebServiceHostFactory" %>
```
<ol start="2">
  <li>файл Service.dll скопировать в папку \bin</li>
  <li>в файл default.aspx в тег body добавить:</li>
</ol>
```html
<a href="DataProvider.svc/Now">Now</a>
```
<ol start="4">
  <li>обновить открытое окно IE; в результате на странице появится ссылка Now; переход по этой ссылке приводит к вызову метода GetTicks и в IE отобразится, например:</li>
</ol>
```xml
<long xmlns="http://schemas.microsoft.com/2003/10/Serialization/">633761956465306263</long>
```
Примечание: если сервис был откомпилирован в режиме (Configuration: Debug), затем подключен к сайту и хоть раз вызван, то в момент повторной компиляции получим сообщение: Unexpected error creating debug information file '....PDB' -- '....pdb: The process cannot access the file because it is being used by another process.'
Решение: в проекте сайта открыть файл web.config и сохранить его; в результате сайт будет перезапущен; доступ к .pdb-файл будет открыт.
