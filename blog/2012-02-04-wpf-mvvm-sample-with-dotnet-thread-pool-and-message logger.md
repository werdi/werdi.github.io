---
title: WPF MVVM sample with .NET Thread pool and message logger
tags: [c#, xaml, wpf, async]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a8fbb994-9ddf-4df1-96a3-83c7bcfe05f2/wpf-mvvm-sample-with-net-thread-pool-and-message-logger?forum=wpf
---
# WPF MVVM sample with .NET Thread pool and message logger
*> is there any way where we can run multiple background threads and synchronize the work done by them*
if you need to syncronize threads, use [ManualResetEvent] (http://msdn.microsoft.com/en-us/library/system.threading.manualresetevent.aspx) or [Barrier] (http://msdn.microsoft.com/en-us/library/system.threading.barrier.aspx) Class and etc.
but if you need to post data from one thread to another, then use [AsyncOperationManager] (http://msdn.microsoft.com/en-us/library/system.componentmodel.asyncoperationmanager.aspx).
an example is [here] (http://social.msdn.microsoft.com/Forums/en-us/winforms/thread/4823af90-e083-4fce-9edf-d9a47700d705/#7caada1f-1f29-429f-9155-b388a983574f)

*> I would like to see in the WPF MVVM sample context given by Bob in the above replies. because winform follows different mechanism.*

with AsyncOperationManager you can write class, which work properly without any changes under all application models (WPF, WinForms, ASP.NET)
 
below is an example for WPF, 
but a Model Class below you can use in a WinForm without any changes.
```xaml
<Window x:Class="WpfApplication4.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Width="325" Height="100" FontSize="16"
  xmlns:app="clr-namespace:WpfApplication4">
  <Window.Resources>
    <app:Model x:Key="model" />
  </Window.Resources>
  <TextBlock Text="{Binding Source={StaticResource model}, Path=Value}" />
</Window>
```
```c#
using System.ComponentModel;
using System.Threading;
using System.Threading.Tasks;
using System.Windows;

namespace WpfApplication4
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }
  }

  class Model : INotifyPropertyChanged
  {
    AsyncOperation Async;
    public event PropertyChangedEventHandler PropertyChanged = delegate { };

    public Model()
    {
      this.Async = AsyncOperationManager.CreateOperation(null);
      Task.Factory.StartNew(Run);   // starts a new thread
    }

    public int Value { get; private set; }

    void Run()
    {
      for (int i = 0; i <= 1000000; i++)
      {
        this.Value = i;
        this.OnPropertyChanged(new PropertyChangedEventArgs("Value"));

        Thread.Sleep(50);  // just for the demo
      }
    }

    void OnPropertyChanged(PropertyChangedEventArgs e)
    {
      SendOrPostCallback cb = state =>
        this.PropertyChanged(this, (PropertyChangedEventArgs)state);
      this.Async.Post(cb, e);
    }
  }
}
```

*> How to collect data from, lets say 5 threads if I write Task.Factory.StartNew(Run) 5 times, individually or wait for all the 5 threads to collect the results till they finish.*

you can collect data in a concurrent bag. and wait threads in another thread.
below is an example.
```c#
using System.ComponentModel;
using System.Threading;
using System.Threading.Tasks;
using System.Windows;
using System.Linq;

namespace WpfApplication4
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }
  }

  class Model : INotifyPropertyChanged
  {
    AsyncOperation Async;
    public event PropertyChangedEventHandler PropertyChanged = delegate { };

    public Model()
    {
      this.Async = AsyncOperationManager.CreateOperation(null);
      this.Results = new System.Collections.Concurrent.ConcurrentBag<int>();

      var r1 = Run(1000, 5);   // starts a new thread
      var r2 = Run(1000, 10);   // starts a new thread
      var r3 = Run(100, 15);   // starts a new thread

      Task.Factory.StartNew(() =>
      {
        WaitHandle.WaitAll(new [] { r1, r2, r3 });   // wait threads
        this.SetValue(this.Results.Sum());
      });
    }

    System.Collections.Concurrent.ConcurrentBag<int> Results;
    public int Value { get; private set; }
    void SetValue(int v)
    {
      this.Value = v;
      this.OnPropertyChanged(new PropertyChangedEventArgs("Value"));
    }

    WaitHandle Run(int count, int time)
    {
      var e = new ManualResetEvent(false);
      Task.Factory.StartNew(() =>
      {
        for (int i = 0; i <= count; i++)
        {
          this.SetValue(i);
          Thread.Sleep(time);  // just for the demo
        }
        this.Results.Add(count);
        e.Set();    // signal
      });
      return e;
    }

    void OnPropertyChanged(PropertyChangedEventArgs e)
    {
      SendOrPostCallback cb = state =>
        this.PropertyChanged(this, (PropertyChangedEventArgs)state);
      this.Async.Post(cb, e);
    }
  }
}
```
 
