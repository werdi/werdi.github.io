---
title: Close and Reopen a WPF Window, possible?
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/780c9f72-68c7-42ad-b6f0-81ed918b3073/close-an-reopen-a-wpf-window-possible-?forum=wpf
---
# Close and Reopen a WPF Window, possible?
*> But what i want to do is: -Be able to close the window (this also works via the Window.Close() Method -Be able to restart the window without restarting the whole application*

try to handle the closing event and simply hide the window. use the show method to open the window later. 
below is an example.
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="Test" Height="300" Width="500">
  <WrapPanel>
    <Button Content="Close" Click="Close_Click" />
    <Button Content="Exit" Click="Exit_Click" />
  </WrapPanel>
</Window>
```
```c#
using System.Windows;
using System.Windows.Input;
using System.Windows.Threading;
using System;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.Closing += new System.ComponentModel.CancelEventHandler(MainWindow_Closing);
    }

    void MainWindow_Closing(object sender, System.ComponentModel.CancelEventArgs e)
    {
      this.Hide();
      e.Cancel = true;
      Delay(TimeSpan.FromSeconds(2), this, wnd => wnd.Show());
    }

    void Delay<T>(TimeSpan ts, T state, Action<T> a)
    {
      var dt = new DispatcherTimer();
      dt.Tick += (s, _) =>
      {
        dt.Stop();
        a(state);
      };
      dt.Interval = ts;
      dt.Start();
    }

    private void Close_Click(object sender, RoutedEventArgs e)
    {
      this.Close();
    }
    private void Exit_Click(object sender, RoutedEventArgs e)
    {
      Application.Current.Shutdown();
    }
  }
}
```