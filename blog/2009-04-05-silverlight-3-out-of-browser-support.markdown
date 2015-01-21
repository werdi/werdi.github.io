---
title: Silverlight 3. Out-of-Browser Support
tags: [out-of-browser, silverlight, xaml]
src: http://cognitex.blogspot.ru/2009/04/silverlight-3-winform.html
---

# Silverlight 3: Out-of-Browser Support

В Silverlight 3 добавлена поддержка Out-of-Browser - возможность запускать приложение вне браузера - определяется разработчиком приложения. Для этого в Visual Studio в Solution Explorer'е надо открыть файл AppManifest.xml (находится в Properties) и добавить в него, например, следующий xml-код:
```xaml
<Deployment xmlns="http://schemas.microsoft.com/client/2007/deployment"
        	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  	<Deployment.Parts>
  	</Deployment.Parts>
  	<Deployment.ApplicationIdentity>
    	<ApplicationIdentity
        	ShortName="Test"
        	Title="Test">
      	<ApplicationIdentity.Blurb>
        	Test</ApplicationIdentity.Blurb>
      	<ApplicationIdentity.Icons>
      	</ApplicationIdentity.Icons>
    	</ApplicationIdentity>
  	</Deployment.ApplicationIdentity>
</Deployment>
```
В результате, после запуска Silverlight приложения в его контекстном меню появится "Install Test element onto this computer ...". При его нажатии появляется диалог "Install application" ... 
Silverlight приложение и метаинформация записывается в папку C:\Users\Имя_пользователя\AppData\LocalLow\Microsoft\Silverlight\Offline\Идентификатор_приложения. 
В Start Menu появится ярлык (в свойствах которого в поле Target указано: "C:\Program Files\Microsoft Silverlight\3.0.40307.0\sllauncher.exe" Идентификатор_приложения).
Просмотр окна, созданного sllauncher.exe, с помощью spyxx.exe показывает наличие следующий иерархии потомков:
```
Shell Embedding
   Shell DocObject View
      Internet Explorer_Server
         ...
```
Т.е. sllauncher.exe создает окно и встраивает в него браузер, в котором запускается Silverlight-приложение.
Удалить Out-of-Browser-приложение можно, если в его контекстом меню выбрать "Remove this application...".
