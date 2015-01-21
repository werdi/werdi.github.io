---
title: Как перехватить трафик ASP.NET Development Server'а
tags: [asp.net, fiddler, testing]
src: http://cognitex.blogspot.ru/2009/04/aspnet-development-server.html
---
# Как перехватить трафик ASP.NET Development Server'а

Ситуация: ASP.NET Development Server работает по адресу http://localhost:3840; требуется перехватить все запросы и ответы сервера. 

Решение: 
<ol>
  <li>во всех адресах заменить "localhost" на "ipv4.fiddler". Например, вместо http://localhost:3840/Provider.svc/Now указать http://ipv4.fiddler:3840/Provider.svc/Now</li>
  <li>запустить <a href="http://www.fiddler2.com/Fiddler2/version.asp">Fiddler</a> (см. Install Fiddler2)</li>
</ol>

Запросы можно увидеть в окне Fiddler'а во вкладке Session Inspector - Raw, а на ответы можно посмотреть в нижней части вкладки, также нажав Raw.

ASP.NET Development Server находится в C:\Program Files\Common Files\microsoft shared\DevServer\9.0\WebDev.WebServer.EXE; параметры передаются через командую строку, например: /port:3840 /path:"С:\Projects\Site" /vpath:"/". 
Сам веб-сервер - это класс Microsoft.VisualStudio.WebHost.Server, который помещен в GAC (WebDev.WebHost.dll). 
Путь к файлу можно выяснить с помощью Process Monitor'а (см. [Sysinternals Process Utilities] (http://technet.microsoft.com/en-us/sysinternals/bb545027.aspx)). Открыть файл в проводнике не получится, но можно его предварительно скопировать с помощью copy в Command Prompt.
