---
title: Silverlight в WinForms
tags: [out-of-browser, silverlight, winforms, xaml]
src: http://cognitex.blogspot.ru/2009/04/silverlight-winforms.html
---
# Silverlight в WinForms
Silverlight можно запустить в WinForms-приложении. Для этого надо: 
<ol>
  <li>Создать Windows Forms Application;</li>
  <li>В References добавить "C:\Program Files\Microsoft Silverlight\3.0.40307.0\npctrl.dll" (появится XcpControlLib);</li>
  <li>В конструктор формы добавить следующий код:</li>
</ol>
```c#
string html = @"
        	<BODY STYLE='margin: 0px; padding: 0px;'>
            	<SCRIPT TYPE='text/javascript'>
            	function onSilverlightError(sender, args)
            	{
                	alert(args.errorMessage + ' ' + args.errorType);
            	}
            	</SCRIPT>
            	<SCRIPT id='_Xaml' type='text/xaml'>
                	<?xml version='1.0' encoding='utf-8' ?>
                	<Canvas
                    	xmlns='http://schemas.microsoft.com/winfx/2006/xaml/presentation'
                    	xmlns:x='http://schemas.microsoft.com/winfx/2006/xaml'>
                        	<TextBlock FontSize='40' Foreground='Red'>Hello, World!</TextBlock>
                	</Canvas>
            	</SCRIPT>
            	<OBJECT ID='_Ag' CLASSID='clsid:DFEAF541-F3E1-4C24-ACAC-99C30715084A' Width='100%' Height='100%'>
                	<PARAM NAME='Background' VALUE='#FFFADF' />
                	<PARAM NAME='onerror' VALUE='onSilverlightError' />
                	<PARAM NAME='Source' VALUE='#_Xaml' />
            	</OBJECT>
        	</BODY>";

WebBrowser wb = new WebBrowser();
wb.Dock = DockStyle.Fill;
wb.DocumentText = html;
wb.Parent = this;
```
Есть одно ограничение: в xaml нельзя использовать, например, Button; получим exception: "ParserError: Unknown element: Button". Причина в том, что Button определен в System.Windows.Controls (System.Windows.dll); Для использования контролов из Silverlight (старше 1) требуется компиляция; в результате будет создан .xap-файл, который необходимо загрузить в Silverlight.
