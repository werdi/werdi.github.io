---
title: Тестируем работу клиента web-сервера
tags: [fiddler, testing, web server, c#]
src: http://cognitex.blogspot.ru/2009/04/web.html
---
# Тестируем работу клиента web-сервера
Протестировать работу клиента веб-сервера можно без самого веб-сервера. Достаточно установить [Fiddler] (http://www.fiddler2.com/Fiddler2/version.asp) (см. Install Fiddler2) - это прокси с AutoResponder'ом. Автором Fiddler'а является Eric Lawrence - сотрудник Microsoft.
Предположим, что по адресу http://getvaluetest.ru расположен сайт, который на любой запрос присылает следующий http-ответ:
```
HTTP/1.1 200 OK
Date: Thu, 16 Apr 2009 08:39:02 GMT
Set-Cookie: name1=5; expires=Tue, 13-Apr-2010, 23:43:04, GMT; path=/; domain=.getvaluetest.ru
Set-Cookie: name2=5; expires=Tue, 13-Apr-2010, 23:43:04, GMT; path=/; domain=.getvaluetest.ru
Content-Length: 6
Content-Type: text/html; charset=windows-1251

привет
```
Сайта не существует, поэтому надо сохранить http-ответ в текстовый файл, например, c:\temp\200.txt, запустить Fiddler, в правой панели стать на AutoResponder и нажать кнопку Add. В левом поле Rule Editor указать http://GetValueTest.ru, а в правом поле - c:\temp\200.txt, нажать Save.
В приложение надо добавить слудующий код:
```c#
HttpResponse hr = new HttpResponse();
// запросить через прокси (Fiddler, который на IP: 127.0.0.1 "слушает" порт 8888)
hr.LoadFrom("http://getvaluetest.ru", new WebProxy(IPAddress.Loopback.ToString(), 8888));
System.Diagnostics.Trace.WriteLine(hr.Trace());
```
Ниже приводятся класс HttpResponse и его методы расширения:
```c#
public class HttpResponse
{
    	public NameValueCollection Headers { get; set; }
    	public string Contnet { get; set; }
    	public Exception Error { get; set; }
}

public static class HttpResponseHelper
{
    	public static void LoadFrom(this HttpResponse res, string url, WebProxy proxy)
    	{
        	HttpWebRequest req = (HttpWebRequest)WebRequest.Create(url);
        	req.Proxy = proxy;
        	req.Method = WebRequestMethods.Http.Get;
        	try
        	{
            	using (HttpWebResponse wres = (HttpWebResponse)req.GetResponse())
            	{
                	if (req.HaveResponse)
                	{
                    	// если в http-ответе не указан charset, то получим: ISO-8859-1
                    	var encoding = Encoding.GetEncoding(wres.CharacterSet);
                    	using (StreamReader sr = new StreamReader(wres.GetResponseStream(), encoding))
                        	res.Contnet = sr.ReadToEnd();

                    	// копируем, потому что HttpWebResponse.Headers.GetValues(string)
                    	// учитывает запятые в значении "Set-Cookie".
                    	res.Headers = new NameValueCollection(wres.Headers);
                	}
            	}
        	}
        	catch (Exception exc)
        	{
            	res.Error = exc;
        	}
    	}

    	public static string Trace(this HttpResponse hr)
    	{
        	if (hr.Error != null)
            	return hr.Error.ToString();

        	StringBuilder sb = new StringBuilder();
        	foreach (string key in hr.Headers.AllKeys)
        	{
            	foreach (string value in hr.Headers.GetValues(key))
                	sb.Append(key).Append(": ").Append(value).Append(Environment.NewLine);
        	}
        	sb.Append(Environment.NewLine);
        	sb.Append(hr.Contnet);
        	return sb.ToString();
    	}
}
```
