---
title: A problem using System.Windows.Forms.WebBrowser object
tags: [c#, winforms, async]
src: https://social.msdn.microsoft.com/Forums/ru-RU/1decbc09-a251-4f72-aec4-7ca19071eed3/a-problem-using-systemwindowsformswebbrowser-object?forum=winforms
---
# A problem using System.Windows.Forms.WebBrowser object
*> I am using System.Windows.Forms.WebBrowser object in order to scan web pages [...] I use it under 2 processes running on the same machine: IIS7 and a windows service*

try using the following code
```c#
using System.Drawing;
using System.Threading;
using System.Windows.Forms;
...
Worker.Start("http://localhost");
...
public class Worker
{
  public static void Start(string url)
  {
    Thread thread = new Thread(() =>
    {
      var ac = new ApplicationContext();
      Application.Idle += delegate
      {
        var thumbnail = new Worker(ac, url);
        thumbnail.Run();
      };
      Application.Run(ac);
    });
    thread.SetApartmentState(ApartmentState.STA);
    thread.Start();
  }
  
  private Worker(ApplicationContext ac, string url)
  {
  _Context = ac;
  _Url = url;
  }
  private bool _Complete = false;
  private ApplicationContext _Context;
  private string _Url;
  
  void Run()
  {
    using (WebBrowser webBrowser = new WebBrowser())
    {
      webBrowser.Size = new Size(1000, 1000);
      webBrowser.ScrollBarsEnabled = false;
      webBrowser.ScriptErrorsSuppressed = true;
      webBrowser.DocumentCompleted += (s, e) =>
        {
          DocumentCompleted(((WebBrowser)s).Document);
          _Complete = true;
        };
      webBrowser.Navigate(_Url);
    
      while (!_Complete)
        Application.DoEvents();
    
        _Context.ExitThread();
    }
  }
  
  void DocumentCompleted(HtmlDocument doc)
  {
    // ...
    System.Diagnostics.Trace.WriteLine(doc.Url);
  }
}
```
*> For a spider, I don't think using a WebBrowser control is a good way to go.*

html spider doesn't helps if the web page content is created dynamically using javascript or jsonp
