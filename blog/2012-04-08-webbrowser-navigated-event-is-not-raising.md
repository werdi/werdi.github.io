---
title: WebBrowser.Navigated Event is not raising
tags: [c#, webbrowser, threading, console]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9cc53ca5-6124-4a1d-a3c5-87efe59806ae/webbrowsernavigated-event-is-not-raising?forum=csharpgeneral
---
# WebBrowser.Navigated Event is not raising
*> the problem raising is that Navigated event never raised.*

use the following: 
```c#
bro.Navigate("http://hotmail.com");
```
instead of bro.Url = new Uri("http://hotmail.com"); 


*> I have tried it on Windows Form, its working great. But not on Console...*

WebBrowser requires a separate STA-thread. 
in a Console application you can do the following:
```c#
using System;
using System.Threading;
using System.Windows.Forms;

namespace ConsoleApplication1
{
  class Program
  {
    static void Main(string[] args)
    {
      var ac = new ApplicationContext();
      var t = new Thread(() =>
      {
        WebBrowser bro = new WebBrowser();
        bro.Navigated += bro_Navigated;
        bro.Navigating += bro_Navigating;
        bro.Navigate("http://hotmail.com");
        Application.Run(ac);
      });
      t.SetApartmentState(ApartmentState.STA);
      t.Start();
      Console.Read();
      ac.ExitThread();
    }

    static void bro_Navigating(object sender, WebBrowserNavigatingEventArgs e)
    {
      Console.WriteLine("Navigating: " + e.Url);
    }
    static void bro_Navigated(object sender, WebBrowserNavigatedEventArgs e)
    {
      Console.WriteLine("Navigated: " + e.Url);
    }
  }
}
```

*> Well i am using [STAThread] on top of my Main function. Is this actually a problem?* 

there is one old issue "[A console application that is based on a STA may delay the release of COM components and may delay the calls to the Finalize methods of the objects that the garbage collector collects] (http://support.microsoft.com/kb/828988#appliesto)" -- this may applied because WebBrowser uses COM components.
