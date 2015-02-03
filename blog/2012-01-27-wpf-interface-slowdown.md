---
title: WPF тормозит интерфейс
tags: [c#, xaml, wpf, async]
src: https://social.msdn.microsoft.com/Forums/ru-RU/ef095386-d375-43f2-b9e6-22be73d71f2a/wpf-?forum=fordesktopru
---
# WPF тормозит интерфейс
*> мне удалось узнать, что процессор занимает "Dispatcher Invoke".*

попробуйте заменить на Dispatcher.BeginInvoke

*> на клике мышкой он не вызывается. в чем тогда смысл замены?*

в том, что [Dispatcher.BeginInvoke] (http://msdn.microsoft.com/ru-ru/library/system.windows.threading.dispatcher.begininvoke.aspx) выполняется асинхронно.
потоки не ждут завершения вызова, как в случае с Dispatcher.Invoke (выполняется синхронно).
в итоге UI не тормозит.

*> Загрузка ЦП 0%. я начинаю ПРОСТО кликать на пустом месте формы и загрузка поднимается до 25%!*

следующий пример что выдает? у меня не более 3.
```c#
using System;
using System.Diagnostics;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Threading;

namespace WpfApplication3
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      var lbl = new Label() { Content = "loading..." };
      this.Content = lbl;
      
      var p = Process.GetCurrentProcess();
      var cpu = new PerformanceCounter("Process", "% Processor Time", p.ProcessName, true);
      
      var count = 0;
      float sum = 0;
      new DispatcherTimer { Interval = TimeSpan.FromSeconds(1), IsEnabled = true }
        .Tick += (s, e) =>
          {
            var v = cpu.NextValue();
            sum += v;
            lbl.Content = count++ + ": process CPU = " + v + "; " + (sum/count);
          };
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication3.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Width="500" FontSize="16" Height="400" >
</Window>
```
