---
title: Публикация значения ПО
tags: [c#, winforms, web service, host]
src: https://social.msdn.microsoft.com/Forums/ru-RU/0c308efa-8757-49d7-b03d-1f6f22067d28/-?forum=desktopprogrammingru
---
# Публикация значения ПО
*> необходимо сделать вариант "публикации" значений для того, что бы другое приложения могло получить эти значения.*

в приложении можно создать WebServiceHost и сервис, который по запросу выдает значения в формате html или json и т.д.
для компиляции примера: 
в Visual Studio в режиме Run as administrator
в свойствах проект указать Application: Target framework = .NET Framework 4
подключить сборки: System.ServiceModel.dll, System.ServiceModel.Web.dll
```c#
using System;
using System.Diagnostics;
using System.IO;
using System.ServiceModel;
using System.ServiceModel.Web;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var sc = new SplitContainer { 
        Parent = this, Dock = DockStyle.Fill, Orientation= Orientation.Horizontal, SplitterDistance = 30 };
      var lbl = new Label { Dock = DockStyle.Fill, Parent = sc.Panel1 };
      
      var count = 0;
      var srv = new Service("http://localhost:8887", str => { lbl.Text = ++count + " " + str; });
      var host = new WebServiceHost(srv, srv.Uri);
      host.Open();
      this.FormClosing += (s, e) => host.Close();

      // test1
      this.Shown += (s, e) => Process.Start(srv.Uri.ToString());

      // test2
      new WebBrowser { Parent = sc.Panel2, Dock = DockStyle.Fill }
        .Navigate(srv.Uri+"test");
    }
}

[ServiceContract, ServiceBehavior(InstanceContextMode = InstanceContextMode.Single)]
public class Service
{
  Action<string> Callback;
  public readonly Uri Uri;
  volatile float _Cpu;

  public Service(string uri, Action<string> cb)
  {
    this.Callback = cb;
    this.Uri = new Uri(uri);
    var p = Process.GetCurrentProcess();
    var cpu = new PerformanceCounter("Process", "% Processor Time", p.ProcessName, true);
    new System.Windows.Forms.Timer() { Interval = 1000, Enabled = true }
      .Tick += (s, e) => _Cpu = cpu.NextValue();
  }
  [OperationContract, WebGet(UriTemplate = "/")]
  public System.ServiceModel.Channels.Message Html()
  {
    this.Callback("/");
    var ret =
      @"<!DOCTYPE html>
        <html>
          <head>
            <META http-equiv='Refresh' content='1; url=' />
            <meta http-equiv='X-UA-Compatible' content='IE=9' />
          </head>
          <body>
            % Processor Time: " + _Cpu + @"
          </body>
        </html>";
    return WebOperationContext.Current.CreateTextResponse(ret, "text/html");
  }
  [OperationContract, WebInvoke(UriTemplate = "/json/", Method="POST")]
  public System.ServiceModel.Channels.Message Json(Stream s)
  {
    var str = new StreamReader(s).ReadToEnd();
    this.Callback("/json/" + str);
    return WebOperationContext.Current.CreateJsonResponse(_Cpu);
  }
  [OperationContract, WebGet(UriTemplate = "/test")]
  public System.ServiceModel.Channels.Message Test()
  {
    var html = @"
          <!DOCTYPE html>
          <html>
            <head>
            <meta http-equiv='X-UA-Compatible' content='IE=9' />
            <script src='http://code.jquery.com/jquery-latest.js' type='text/javascript'></script>
            <script type='text/javascript'>
            function result (d, status) { $('#resid').html(status + ': ' + d); }
            $(document).ready(function () {
              $('#frmid').submit(function () {  
                $.post('/json/', {'type':'cpu'}, result);
                return false;
              });
            });
            </script>
            </head>
            <body>
              <form method='post' action='' id='frmid'>
                <input type='submit' /></form>    
              <div id='resid'></div>
            </body>
          </html>";
      return WebOperationContext.Current.CreateTextResponse(html, "text/html");
    }
  }
}
```
*> А можно сделать что бы данные можно было прочитать используя WMI?*

если надо через WMI получить значения счетчиков производительности, то см. [Performance Counter Classes] (http://msdn.microsoft.com/en-us/library/windows/desktop/aa392738(v=vs.85).aspx); для работы с WMI из C# см. [пример] (http://social.msdn.microsoft.com/Forums/en-us/wpf/thread/6bc619b0-bb7f-481d-87a3-edd77ecd5dd8/#d034bd5e-760b-4397-8afa-ee4c4e42fbd6).
если надо предоставлять значения через WMI, то см. [Writing coupled WMI providers using WMI.NET Provider Extension 2.0] (http://msdn.microsoft.com/en-us/library/cc268228.aspx)

*> Если вы хотите чтобы вторая программа запрашивала данные у первой наподобии того, как вы запрашиваете данные у системы, используя WMI - то это врядли выйдет.*

см. MSDN Blogs > Windows Management Infrastructure Blog > [Writing WMI provider in a day] (http://blogs.msdn.com/b/wmi/archive/2009/06/19/writing-wmi-provider-in-a-day.aspx);
и [WMI Provider Extensions in .NET 3.5] (http://www.codeproject.com/Articles/25783/WMI-Provider-Extensions-in-NET-3-5)
