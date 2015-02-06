---
title: Универсальное приложение
tags: [c#, threading, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/534c790d-01aa-4652-905b-89ce02e162f9/-?forum=fordesktopru
---
# Универсальное приложение
*> При запуске показывается окно с выбором [...] При запуске выбранной, в нее передаются необходимые данные [...] Как это можно реализовать?  [...] Или почитать где?*

см. MEF, [Managed Extensibility Framework Overview] (http://msdn.microsoft.com/ru-ru/library/dd460648.aspx)

*> это, как понял, облегченный вариант подключенных библиотек.*

да

*> можно выполнить процедуру, которая в библиотеке не определена интерфейсом?*

да. можно в рантайме создать код, выполнить компиляцию и вызвать. пример [здесь] (http://social.msdn.microsoft.com/Forums/en-US/fordesktopru/thread/b741c190-cbaa-4349-94a3-7ea112112ee4/#0ec1a63c-f35c-4090-8084-4829161cb59e).

*> каждая программа имеет свое отдельное окно и элементы... к ним можно обращаться из самой программы или необходимо из главной программы-родителя?*

возможны разные сценарии. даже неуправляемую windows-программу можно встроить в свое .net-приложение.

*> программа имеет несколько окон, свои классы и модули. Как к ним после подключения обратиться, если программа не будет знать к какой процедуре обращаться?*

существуют браузеры .net-сборок. например: [reflector] (http://en.wikipedia.org/wiki/.NET_Reflector) и [ilspy] (http://wiki.sharpdevelop.net/ILSpy.ashx) -- позволяют восстановить код сборок.
зная тип и имя метода, можно создать экземпляр известного типа и вызвать метод.
```c#
using System;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var t = Type.GetType("WindowsFormsApplication3.Test");
      var o = Activator.CreateInstance(t);
      var m = t.GetMethod("Run");
      var r = m.Invoke(o, new [] { "world" });
      System.Diagnostics.Trace.WriteLine(r);
    }
  }


  class Test
  {
    public string Run(string p)
    {
      return "hello, " + p;
    }
  }
}
```
*> найти подключаемую программу и запустить ее... ну и передать нужную информацию при запуске, переменную*

app1 запускает процесс с app2 и передает параметры через command line:

[app1.exe]
```c#
using System.Diagnostics;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      new Button { Parent = this, Text = "Start" }
        .Click += (s, e) =>
           Process.Start(
             @"C:\Projects\app2.exe", "hello world");
    }
  }
}
```
[app2.exe]
```c#
using System;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      new RichTextBox
      {
        Parent = this,
        Dock = DockStyle.Fill,
        Text = String.Join("; ", Environment.GetCommandLineArgs())
      };
    }
  }
}
```
другие способы передачи данных: [WM_COPYDATA] (http://msdn.microsoft.com/en-us/library/windows/desktop/ms649011(v=vs.85).aspx), Message Queuing ([MSMQ] (http://msdn.microsoft.com/en-us/library/windows/desktop/ms711472(v=vs.85).aspx)), REST (см. [здесь] (http://social.msdn.microsoft.com/Forums/ru-RU/fordesktopru/thread/07bfe3cc-e126-4991-b9d7-8718f4e8a76f)) 
и т.д. [здесь] (http://www.gotdotnet.ru/forums/2/22271/110727/#post110727)

*> создается класс и отдельный поток для класса. В классе мы ожидаем ответа сервера*

следующий код создает отдельный поток и ждет запросы клиентов:
```c#
var host = new WebServiceHost(typeof(Service), new Uri(url));
host.Open();
```
клиенты обращаются к серверу так:
```c#
var c = new WebClient();
var res = c.DownloadString(url);      // или DownloadData и т.д.
```
другие примеры: [здесь] (http://social.msdn.microsoft.com/Forums/ru-RU/aspnetru/thread/b317d235-0570-4d4e-b87e-04a771b4f022#315516c8-7488-470f-a629-eb7290f627ce) и [здесь] (http://social.msdn.microsoft.com/Forums/ru/fordataru/thread/bf8871c1-0cd2-4cad-a80e-32d1083a1dd0#827f1e46-3034-4196-bc67-1ef79c030e93)

если надо выполнить обращение к серверу в отдельном потоке, а затем вывести ответ на UI:
```c#
Task.Factory.StartNew(() =>    // выполняется в отдельном потоке
{
  var c = new WebClient();
  return c.DownloadData(url);
})
.ContinueWith(t =>                  // выполняется в основном потоке
{
    var r = t.Result;
    // здесь код для вывода r на ui.
}, 
TaskScheduler.FromCurrentSynchronizationContext());
```
