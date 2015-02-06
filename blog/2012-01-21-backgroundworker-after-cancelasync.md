---
title: How can I wait for a BackgroundWorker to end after calling CancelAsync?
tags: [c#, winforms, async]
src: https://social.msdn.microsoft.com/Forums/ru-RU/14679d00-3d46-4de4-be87-e66c196f0638/how-can-i-wait-for-a-backgroundworker-to-end-after-calling-cancelasync?forum=csharpgeneral 
---
# How can I wait for a BackgroundWorker to end after calling CancelAsync?
*> I've a BackgroundWorker [...] But I get an ObjectDisposeException at run time when I close the form*

handle FormClosing event for call CancelAsync
below is an example
```c#
using System.ComponentModel;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var pb = new ProgressBar { 
        Dock = DockStyle.Top, Parent = this, Maximum = 1000 };

      var w = new BackgroundWorker();
      w.WorkerSupportsCancellation = true;
      w.WorkerReportsProgress = true;
      w.ProgressChanged += (s, e) => pb.Value = e.ProgressPercentage;
      w.DoWork += (s, e) => this.DoWork(w);
      
      this.FormClosing += (s, e) => w.CancelAsync();
      w.RunWorkerAsync();
    }
  
    void DoWork(BackgroundWorker w)   
    {
      for (int i = 0; i <= 1000; i++)
      {
        if(w.CancellationPending)
          break;

        w.ReportProgress(i);
        Thread.Sleep(100);  // for test
      }
    }
  }
}
```
*> Malobukv, this isn't a solution, because ddlyProgrammer already call the CancelAsync method in the FormClosing event.*

sure, that call of the CancelAsync method in the FormClosing event is not enough.
also you should check out the CancellationPending property, as in the example above.

*> The following is not a good practice. w.ProgressChanged += (s, e) => [...] How do you unsubscribe? You cannot*

the code above is just an example.
but even in the real code we should never forget the [KISS principle] (http://en.wikipedia.org/wiki/KISS_principle)

*> First, disavow yourself of the idea that you need to "wait" for the background worker. Your UI code should be event driven*

Thread.Sleep Method is used in example above just for the demo. so "// for test" above means "// just for the demo".

*> The reason why you do not want to update your UI more than roughly 20-30 times per second is that your video display doesn't update faster than that.*

there is another reason: data transferring from the worker thread to the UI-thread goes through the Windows Message Queue. if the worker thread sends a lot of messages then this negatively affects UI performance. to avoid this we could use a Stopwatch. below is an example:
```c#
using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var lbl = new Label { Parent = this, Dock = DockStyle.Fill };
      
      var cts = new CancellationTokenSource();
      this.FormClosing += delegate { cts.Cancel(); };

      Action<int> notify = v =>       // ui-thread
        lbl.Text = v.ToString();

      var s = Stopwatch.StartNew();
      Action<int> progress = v =>  // worker thread
      {
        if (s.ElapsedMilliseconds < 100) return;
        s.Restart();
        if(cts.IsCancellationRequested == false)
          this.BeginInvoke(notify, v);
      };
      Task.Factory.StartNew(() =>  // worker thread
      {
        for (int i = 0; i < 1000; i++)
        {
          Thread.Sleep(10);  // just for the demo
          progress(i);
        }
      });
    }
  }
}
```
