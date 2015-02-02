---
title: Некорректное отображение окна WPF
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/7b4b2f29-ac4e-407d-a880-678fdce0f56d/-wpf?forum=fordesktopru
---
# Некорректное отображение окна WPF
*> Проходит время и из стека убираются кнопки. Когда их не осталось, таймер останавливается и окно снова скрывается (Collapsed). [...]  следующий показ будет "обрезан" ...*

проверил. окно выводится нормально.
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="500" Width="500" FontSize="16" Visibility="Collapsed"
  xmlns:app="clr-namespace:WpfApplication1"  SizeToContent="WidthAndHeight">
  <Grid>
    <MediaElement Name="MsxOEM"></MediaElement>
    <StackPanel Name="StackAll"></StackPanel>
  </Grid>
</Window>
```
```c#
using System;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Threading;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      var dir = 1;
      var dt = new DispatcherTimer { Interval = TimeSpan.FromMilliseconds(500) };
      dt.Tick += (s, e) =>
      {
        switch (dir)
        {
          case 1:
            this.Visibility = System.Windows.Visibility.Visible;
            StackAll.Children.Add(new Button { Content = "b" + StackAll.Children.Count, Width = 200 });
            if (StackAll.Children.Count == 3)
                dir = -1;
            break;
          case -1:
            StackAll.Children.RemoveAt(0);
            if (StackAll.Children.Count == 0)
            {
              dir = 1;
              this.Visibility = System.Windows.Visibility.Collapsed;
            }
            break;
        }
      };
      dt.Start();
    }
  }
}
```
