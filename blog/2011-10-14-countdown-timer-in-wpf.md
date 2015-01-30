---
title: Countdown Timer in WPF
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/25705cf9-693b-4c15-a549-ad198986406e/countdown-timer-in-wpf?forum=wpf
---
# Countdown Timer in WPF
```c#
using System;
using System.Windows;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }
    private void Button_Click(object sender, RoutedEventArgs e)
    {
      Countdown(15, TimeSpan.FromSeconds(1), cur => tb.Text =  cur.ToString() );
    }
    void Countdown(int count, TimeSpan interval, Action<int> ts)
    {
      var dt = new System.Windows.Threading.DispatcherTimer();
      dt.Interval = interval;
      dt.Tick += (_, a) =>
      {
        if (count-- == 0)
          dt.Stop();
        else
          ts(count);
      };
      ts(count);
      dt.Start();
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication1.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    Width="400" Height="400"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <Grid>
    <TextBox x:Name="tb" Width="200" Height="40" />
    <Button Content="Start" Click="Button_Click" Width="150" Height="40" Margin="0,100,0,0" />
  </Grid>
</Window>
```
*> If you had some requirement where the timer accuracy was critical then using a storyboard is better.*
could you please provide an example code with countdown timer based on storyboard?
as i know, storyboard runs on the UI thread. 
if the timing accuracy required then we can use stopwatch:
```c#
void Countdown(int count, int interval, Action<int> callback)
{
  var sw = new Stopwatch();
  Action<int> wait = ts => 
  {
    sw.Start();
    while(sw.ElapsedMilliseconds < ts);
    sw.Reset();
  };
  Task.Factory.StartNew(() =>
  {
    var cur = count;
    do
    {
      this.Dispatcher.BeginInvoke(callback, cur);
      wait(interval);
    }
    while (cur-- != 0);
  });
}
...
void Button_Click(object sender, RoutedEventArgs e)
{
  Countdown(10, 1000, cur => tb.Text = cur.ToString());
}
```
