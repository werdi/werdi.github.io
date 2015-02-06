---
title: C# Как управлять progressBar из другой формы?
tags: [c#, winforms, components]
src: https://social.msdn.microsoft.com/Forums/ru-RU/849c26b3-3ae8-47bf-a649-f2d9256fca07/c-progressbar-?forum=fordesktopru
---
# C# Как управлять progressBar из другой формы?
*> Есть несколько форма. [...] progressBar только на «Главной» форме [...] на разных формах вызывают продолжительные вычисления. [...] Прогресс их вычислений должен фиксироваться с помощью progressBar*
```c#
using System;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var pb = new ProgressBar { Parent = this, Dock = DockStyle.Top, 
        Maximum = 100 };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Start", delegate 
      {
        var dlg = new Dialog(pb);
        dlg.ShowDialog();
      });
    }
  }
  
  class Dialog : Form
  {
    public Dialog(ProgressBar pb)
    {
      this.Shown += (s, e) => this.Run(pb);
    }
  
    private void Run(ProgressBar pb)
    {
      Action<int> cb = v => pb.Value = v;
      var end = false;
      this.FormClosing += (s, e) => end = true;
      ThreadPool.QueueUserWorkItem(s => 
      {
        for(int i=0; i <= 100; i++)
        {
          if(end) break;
          Thread.Sleep(100);   // for test
          pb.BeginInvoke(cb, i);
        }                
      });
    }
  }
}
```
другой вариант:
```c#
using System.Windows.Forms;
using System;
using System.Threading.Tasks;
using System.Linq;
using System.Threading;

namespace WindowsFormsApplication4
{
  interface INotifyUser
  {
    void Progress(int value);
  }

  public partial class Form1 : Form, INotifyUser
  {
    public Form1()
    {
      _ProgressBar = new ProgressBar { Parent = this, Dock = DockStyle.Top, Maximum = 100 };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Start", delegate 
      {
        var dlg = new Dialog();
        dlg.ShowDialog();
      });
    }

    ProgressBar _ProgressBar;

    public void Progress(int value)
    {
      if(this.InvokeRequired)
        this.BeginInvoke(new Action<int>(v => this.Progress(v)), value);
      else
        _ProgressBar.Value = value;
    }
  }
  
  class Dialog : Form
  {
    public Dialog()
    {
      var nu = Application.OpenForms.OfType<INotifyUser>().FirstOrDefault();
      var cts = new CancellationTokenSource();
      this.Shown += (s, e) => this.Run(nu, cts.Token);
      this.FormClosing += (s, e) => cts.Cancel();
    }
  
    void Run(INotifyUser nu, CancellationToken ct)
    {
      Task.Factory.StartNew(() => 
      {
          for(int i=0; i <= 100; i++)
          {
            if(ct.IsCancellationRequested) break;
            Thread.Sleep(100);  // for test
            nu.Progress(i);
          }
      });
    }
  }
}
```
*> ProgressBar1 кинул на интерфейс и вызываю с любой формы. Так можно к любому контролу обращаться.А в C# такого решения нет?*
```c#
public partial class Form1 : Form
{
  public ProgressBar Progress { get; set; }

  public Form1()
  {
    this.Progress = new ProgressBar { Parent = this, Dock = DockStyle.Top };
  }
}
```
к свойству Progress (т.е. к контролу ProgressBar) можно обращаться извне.

если сделать можно просто, то для чего усложнять, как в предыдущих примерах? 
см. msdn: "[Связность и увязка] (http://msdn.microsoft.com/ru-ru/magazine/cc947917.aspx)"

*> А Вы могли бы пример дать?*
```c#
using System.ComponentModel;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public ProgressBar ProgressBar;

    public Form1()
    {
      var pb = new ProgressBar { Parent = this, Dock = DockStyle.Top, Maximum = 100 };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Start", delegate
      {
        var dlg = new Dialog();
        dlg.Progress += (s, e) => pb.Value = e.ProgressPercentage;
        dlg.ShowDialog();
      });
    }
  }

  class Dialog : Form
  {
    public Dialog()
    {
      this.Worker = new BackgroundWorker();
      this.Worker.WorkerSupportsCancellation = true;
      this.Worker.WorkerReportsProgress = true;
      this.Worker.ProgressChanged += (s, e) => this.Progress(this, e);
      this.Worker.DoWork += (s, e) => this.DoWork();
      this.FormClosing += (s, e) => this.Worker.CancelAsync();
      this.Worker.RunWorkerAsync();
    }

    BackgroundWorker Worker;
    public event ProgressChangedEventHandler Progress = delegate { };

    void DoWork()   // выполняется в отдельном потоке
    {
      for (int i = 0; i <= 100; i++)
      {
        if(this.Worker.CancellationPending)
          break;

        this.Worker.ReportProgress(i);
        Thread.Sleep(100);  // for test
      }
    }
  }
}
```
*> Теперь в любом месте «trees», в любой функции или процедуре мы можем работать с с progressBar1 из «Главной» формы. [...] Этим методом, можно обращаться к любому контролу и не мучаться.*

связать две формы в микропроекте можно как угодно, но если планируете его развивать, то завтра: "не мучаться" превратится в "ад", т.к. части кода сильно зависят друг от друга, тестирование и  рефакторинг станут практически невозможны.
