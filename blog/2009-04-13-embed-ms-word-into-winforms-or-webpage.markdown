---
title: Как встроить Word в WinForms или веб-страницу
tags: [embed, office, web, winforms]
src: http://cognitex.blogspot.ru/2009/04/word-winforms.html
---
# Как встроить Word в WinForms или веб-страницу
Word, Excel, Visio, PowerPoint можно встроить в Windows Forms Application или веб-страницу с помощью DSOFramer (это ActiveX), который входит в состав [Microsoft Developer Support Office Framer Control 1.3 Sample (KB 311765)] (http://www.microsoft.com/downloads/details.aspx?FamilyId=CE2CA4FD-2169-4FAC-82AF-770AA9B60D77&displaylang=en).

После установки примеров (C:\DsoFramer) надо: 
<ol>
  <li>В Visual Studio создать новый проект (Windows Forms Application)</li>
  <li>В References добавить ссылку на C:\DsoFramer\dsoframer.ocx (в результате будет создан файл Interop.DSOFramer.dll; в References появится DSOFramer)</li>
  <li>Добавить в проект класс:</li>
</ol>
```c#
public class Fcc : AxHost
{
    	public Fcc()
        	: base("00460182-9E5E-11d5-B7C8-B8269041DD57")
    	{
    	}
}
```
<ol start="4">
  <li>Код конструктора Form1 заменить на следующий:</li>
</ol>
```c#
this.Size = new Size((int)(SystemInformation.WorkingArea.Width * .90f), 400);
this.Shown += delegate
{
    	var fcc = new Fcc();
    	fcc.Dock = DockStyle.Fill;
    	fcc.Parent = this;

    	var ocx = fcc.GetOcx() as FramerControl;
    	ocx.Titlebar = false;
    	ocx.Toolbars = false;
    	ocx.Menubar = false;
    	ocx.EventsEnabled = true;

    	ocx.Open(@"C:\Temp\Test.docx", Missing.Value, Missing.Value, Missing.Value, Missing.Value);
    	ocx.Activate();
};
```
Метод ocx.Open также можно вызвать следующим способом:
```c#
typeof(_FramerControl).GetMethod("Open").Invoke(ocx, new object[] { @"C:\Temp\Test.docx", Missing.Value, Missing.Value, Missing.Value, Missing.Value });
```
<ol start="5">
  <li>Заменить "C:\Temp\Test.docx" на путь к файлу с расширением: .docx | .doc | .pptx | ... Нажать F5.</li>
</ol>

Чтобы самому не создавать наследника AxHost, можно сделать следующее: 
<ol>
  <li>открыть Form1.cs в дизайнере (= в контекстном меню выбрать View Designer)</li>
  <li>открыть Toolbox (Ctrl+W, X)</li>
  <li>в контекстном меню Toolbox'а выбрать Choose Items... - COM Components</li>
  <li>в списке найти DSO Framer Control Object, включить checkbox</li>
  <li>DSO Framer Control Object перетащить с Toolbar'а и сбросить над формой</li>
</ol>

В результате будет создан файл AxInterop.DSOFramer.dll, а в References появится AxDSOFramer. 

Пример встраивания офисных документов в веб-страницу находится в папке C:\DsoFramer\Samples\WebTest

Подробности использования DSOFramer приведены в статье ["Visual C++ ActiveX Control for hosting Office documents in Visual Basic or HTML"] (http://support.microsoft.com/kb/311765/EN-US/).
