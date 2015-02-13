---
title: Thread problem
tags: [c#, winforms, async, web service, html]
src: https://social.msdn.microsoft.com/Forums/ru-RU/4823af90-e083-4fce-9edf-d9a47700d705/thread-problem?forum=winforms
---
# Thread problem
*> I am working on a Winform WCF application client and have heard that I should always use the UI thread to create and manipulate winforms and controls.*

if you are using WebServiceHost then nothing special is needed for the data transfer to the UI.
below is an example.
```c#
using System;
using System.Net;
using System.ServiceModel;
using System.ServiceModel.Web;
using System.Threading;
using System.Windows.Forms;
using System.Threading.Tasks;
using System.Diagnostics;
using System.IO;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var rtb = new RichTextBox { Parent = this, Dock = DockStyle.Fill };
      var uri = new Uri("http://localhost:8889");
      this.TopMost = true;

      this.Shown += delegate
      {
        var host = new WebServiceHost(typeof(Service), uri);
        host.Open();
        this.FormClosing += delegate { host.Close(); };

        // start browser; invoke Service.Html()
        Process.Start(uri + "html");
      };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Test", (s, e) =>
        new WebClient().DownloadStringAsync(new Uri(uri, "test")) );
    }
  }

  [ServiceContract]
  public class Service
  {
    public RichTextBox Result { get; set; }

    public Service()
    {
      this.Result = Application.OpenForms[0].Controls[0] as RichTextBox;
    }

    [OperationContract, WebGet(UriTemplate = "/test/")]
    public string Test()
    {
      this.Result.AppendText(Thread.CurrentThread.ManagedThreadId + "; ");
      return "ok";
    }

    [OperationContract, WebGet(UriTemplate = "/html/")]
    public Stream Html()
    {
      WebOperationContext.Current.OutgoingResponse.ContentType = "text/html";
      return StringHelper.ToStream(@"
        <body>
          <form action='/test/' method='get' target='frm'>
            <input type='submit' value='test' />
          </form>
          <iframe name='frm' />
        </body>");
    }
  }

  public static class StringHelper
  {
    public static Stream ToStream(this string s)
    {
      var bytes = System.Text.Encoding.Default.GetBytes(s);
      var ret = new MemoryStream();
      ret.Write(bytes, 0, bytes.Length);
      ret.Position = 0L;
      return ret;
    }
  }
}
```
*> ... To make it easy to make sure if we are on the main thread a dummy form that is never shown is created at startup. When I need to check if Im on the correct thread for example create a new winform I just have to check dummyForm.invokeRequired. ...*

dummy form is not required, as well as checking of InvokeRequired and BeginInvoke are not needed.
all data transfer among threads you can do with the AsyncOperationManager.
below is an example.
```c#
using System;
using System.ComponentModel;
using System.Drawing;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var cts = new CancellationTokenSource();
      this.Size = new System.Drawing.Size(400, 600);
      this.AutoScroll = true;
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Start1", (s, e) => 
      {
        var lbl = new Label { Dock = DockStyle.Top, Height = 20, Parent = this };
        var index = this.Controls.Count;
        Test1(val => 
          lbl.Text = "control=" + index + " result={" + val + "}", cts.Token);
      });

      this.Menu.MenuItems.Add("Start2", (s, e) => 
      {
        Action cb = val => 
        {
          var lbl = new Label { Dock = DockStyle.Top, Height = 20, Parent = this, 
              BackColor = Color.FromKnownColor(KnownColor.Info) };
          var index = this.Controls.Count;
          lbl.Text = "control=" + index + " result={" + val + "}";
        };
        Test2(AsyncOperationManager.CreateOperation(cb), 0, cts.Token);
      });

      this.Menu.MenuItems.Add("Stop", (s, e) => 
      {
        this.Menu.MenuItems.Clear();
        cts.Cancel();
      });
      
      this.FormClosing += (s, e) => cts.Cancel();
    }

    void Test1(Action fn, CancellationToken ct)
    {
      // create a 'bridge' to the caller thread
      var ao = AsyncOperationManager.CreateOperation(null);
      new Thread(state =>
      {
        SendOrPostCallback cb = data => // runs in the caller thread
          fn((string)data);
        var rnd = new Random();
        for (int i = 0; i < 100; i++)
        {
          if(ct.IsCancellationRequested) break;
          // post data to the caller thread
          ao.Post(cb, 
            "value=" + i + "; thread=" + Thread.CurrentThread.ManagedThreadId); 
          Thread.Sleep(rnd.Next(100, 1000));  // for test
        }
      })
      .Start();  
    }

    void Test2(AsyncOperation ao, int index, CancellationToken ct)
    {
      new Thread(() => 
      {
        if(ct.IsCancellationRequested) return;
        var fn = ao.UserSuppliedState as Action;
        var data = "value=" + index + 
          "; thread=" + Thread.CurrentThread.ManagedThreadId;
        ao.Post(str => 
          fn((string) str),   // runs in the caller thread
          data);
        Thread.Sleep(2000);
        Test2(ao, index+1, ct);
      })
      .Start();
    }
  }
}
```
![img1](http://social.microsoft.com/Forums/getfile/37970/)

*> So what I need here is to make sure that I always creates a form(also the dialog form) on the mainUI and the question is then how to know this?*

see [AsyncOperationManager](http://msdn.microsoft.com/en-us/library/system.componentmodel.asyncoperationmanager.aspx) description: "... class, which ensures that the client's event handlers are called on the proper thread".  
there is an example of using. also take a look at my example above.

*> I need a simple solution to this so SynchronizationContext will be my first try. AsyncOperationManager seems to contain a SynchronizationContext so I supose that the effect is the same.*

there are various derived types of SyncronizationContext:
- WindowsFormsSynchronizationContext (for WinForms)
- DispatcherSynchronizationContext (for WPF)
- etc.
you should use proper context for different types of applications or simply use the AsyncOperation as mentioned above.
why do so few people use AsyncOperation? i think due to the lack of proper documentation with enough examples.

*> The SynchronizationContext do not hold any threadId? With invokeRequired we would type somthing like this :if (_dummyForm.InvokeRequired) ...*

WindowsFormsSynchronizationContext holds referen to the internal MarshalingControl.
so no need to use _dummyForm.

*> But this is not correct in a winform application? Should I instead use : _threadSyncContext = WindowsFormsSynchronizationContext.Current;*

you should use AsyncOperation as metioned above.
so instead of _threadSyncContext = WindowsFormsSynchronizationContext.Current;
use the following:
```c#
_threadSyncContext = AsyncOPerationManager.CreateOperation(null);
```
also remove  if(Thread.CurrentThread.ManagedThreadId != _threadSyncContext...ManagedThreadId)
and also use .Post instead of .Send

*> I have no tried this but I get the followin errors when building*

1) start Visual Studio in mode "Run as Administrator".
2) open Project Properties - Application and in 'Target Framework' select '.NET Framework'

*> I do also think that its far to little information on internet about how SynchronizationContext works*

there are browsers of .NET-assembly: Reflector, ILSpy, etc.
so you can open mscorlib.dll and have a look at the source code of the SynchronizationContext
   
*> how to choose the right kind of SynchronizationContext for your application*
    
simply use the [AsyncOperationManager.CreateOperation Method](http://msdn.microsoft.com/en-us/library/system.componentmodel.asyncoperationmanager.createoperation), which creates the proper SynchronizationContext.
```c# 
AsyncOperation ao = AsyncOperationManager.CreateOperation(null);
``` 
AsyncOperation is itself a wrapper around the right kind of SynchronizationContext.

*> I know that WindowsFormsSynchronizationContext will be the SynchronizationContext I get from AsyncOperationManager.SynchronizationContext when in a winform application, bit what if I am in another form a application?*

if you asking about another form in same app, then the following code returns the same result.
```c#  
AsyncOperationManager.SynchronizationContext.GetHashCode()
```    
if you asking about class portaility, then take a look at the [example](http://social.msdn.microsoft.com/Forums/en-us/csharpgeneral/thread/8019c463-8edb-4280-a628-5e4e81a9225f#fcededae-a0f4-428f-b16a-dc64fe2cb41d). there is a class (work on a different thread and reports a progress to the UI thread). class is portable without any changes among different types of apps: WinForms and WPF.

*> When using AsyncOperationManager.SynchronizationContext in a Winform application I will get a WindowsFormsSynchronizationContext, but what will I get in a WPF application or an regular Consol application?*

the following code:
```c# 
AsyncOperationManager.SynchronizationContext.GetType().FullName;
```   
gives the following results:
``` 
WinForms App: System.Windows.Forms.WindowsFormsSynchronizationContext
WPF: System.Windows.Threading.DispatcherSynchronizationContext
Console App: System.Threading.SynchronizationContext  
ASP.NET App: System.Web.AspNetSynchronizationContext
etc.
```   
though this doesn't matter. everything works seamlessly. 
i'm wondering which version of Visual Studio do you have?
