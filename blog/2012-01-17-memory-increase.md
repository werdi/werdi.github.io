---
title: Memory Increase
tags: [c#, xaml, wpf, async]
src: https://social.msdn.microsoft.com/Forums/ru-RU/23e87946-353d-44c2-8594-aa844ad7f30e/memory-increase?forum=wpf 
---
# Memory Increase
*> So, how can i create a separate thread to update the textblock per 10 miliseconds without memory increasingperiodically.*

try using the following example
```xaml
<Window x:Class="WpfApplication3.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="300" Width="300">
  <Grid>
    <TextBlock Text="{Binding Test}" />
  </Grid>
</Window>
```
```c#
using System;
using System.ComponentModel;
using System.Windows;
using System.Windows.Threading;

namespace WpfApplication3
{
  public partial class MainWindow : Window, INotifyPropertyChanged
  {
    public MainWindow()
    {
      InitializeComponent();

      this.DataContext = this;

      int n = 0;
      var dt = new DispatcherTimer() { Interval = TimeSpan.FromMilliseconds(10) };
      dt.Tick += (s, e) =>
      {
        if (n++ > 20000) n = 0;
        this.Test = n.ToString();
      };
      this.Loaded += (s, e) =>   dt.Start();
    }

    public string Test
    {
      get { return _Test; }
      set
      {
        _Test = value;
        this.PropertyChanged(this, new PropertyChangedEventArgs("Test"));
      }
    }
    string _Test;

    public event PropertyChangedEventHandler PropertyChanged = delegate { };
  }
}
```
