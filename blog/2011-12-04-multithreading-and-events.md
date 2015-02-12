---
title: Multithreading and events
tags: [c#, xaml, wpf, winforms, async]
src: https://social.msdn.microsoft.com/Forums/ru-RU/8019c463-8edb-4280-a628-5e4e81a9225f/multithreading-and-events?forum=csharpgeneral
---
# Multithreading and events
*> I have a class (lets call it MyClass), its purpose is to do some work on a different thread and report its progress. [...] With all this crazy code aside, what is the proper way to do what I'm trying to accomplish?*
```c#
using System;
using System.ComponentModel;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApplication9
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var pb = new ProgressBar {Parent = this, Dock = DockStyle.Top, Height = 30 };
      var mc = new MyClass();
      mc.ProgressChanged += i => pb.Value = i;
      mc.Run("...");
    }
	}

	public class MyClass
	{
	  public event Action<int> ProgressChanged = delegate { };

	  public void Run(string some)
	  {
      SendOrPostCallback cb = state => ProgressChanged((int)state);
      ParameterizedThreadStart fn = state =>
      {
        var ao = state as AsyncOperation;
        var data = ao.UserSuppliedState as string;
        for (int i = 0; i <= 100; i++)
        {
          ao.Post(cb, i);
          Thread.Sleep(50);
        }
      };
      new Thread(fn).Start(AsyncOperationManager.CreateOperation(some));
    }
  }
}
``` 
the most important thing is that you can use MyClass without any changes 
in a WPF app.  
```xaml
<Window x:Class="WpfApplication8.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="250" Width="700" FontSize="16">
  <Grid>
    <ProgressBar x:Name="pb" Height="30" />
  </Grid>
</Window>
```
```c#
using System;
using System.ComponentModel;
using System.Threading;
using System.Windows;

namespace WpfApplication8
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.Loaded += (s, e) => 
      {
        var mc = new MyClass();
        mc.ProgressChanged += i => pb.Value = i;
        mc.Run("...");
      };
    }
	}

	class MyClass
	{
    public event Action<int> ProgressChanged = delegate { };

    public void Run(string some)
    {
      SendOrPostCallback cb = state => ProgressChanged((int)state);
      ParameterizedThreadStart fn = state =>
      {
        var ao = state as AsyncOperation;
        var data = ao.UserSuppliedState as string;
        for (int i = 0; i <= 100; i++)
        {
          ao.Post(cb, i);
          Thread.Sleep(50);
        }
      };
      new Thread(fn).Start(AsyncOperationManager.CreateOperation(some));
    }
  }
}
``` 
p.s.
 instead of the following:
```c#    
ParameterizedThreadStart fn = state => { /* ... */ };
new Thread(fn).Start(AsyncOperationManager.CreateOperation(some));
```
you can use this one:
```c#   
WaitCallback fn = state => { /* ... */ };
ThreadPool.QueueUserWorkItem(fn, AsyncOperationManager.CreateOperation(some));
```