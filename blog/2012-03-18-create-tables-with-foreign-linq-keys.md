---
title: Создание таблиц с внешними ключами LINQ
tags: [c#, xaml, wpf, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/4bfe605b-3d9c-445d-a837-1176c257f29a/-linq?forum=programminglanguageru
---
# Создание таблиц с внешними ключами LINQ
*> есть грид на MainWindow, но событие подключения к БД и заполнения грида находится на другой форме. Как сделать чтобы при клике на другой форме заполнялся грид?*

ниже пример способа, при котором части приложения сильно связаны.
чтобы снизить связанность можно использовать System.ComponentModel.Composition см. [здесь] (http://social.msdn.microsoft.com/Forums/ru-RU/programminglanguageru/thread/e08c8717-4acc-4559-a0db-3e8ae5f485df/#7a4ad1fb-1fa8-4cab-a57c-e9c9973150bb)
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="500" Width="500">
  <StackPanel>
    <Button Content="dialog" HorizontalAlignment="Left" VerticalAlignment="Top" Click="Button_Click" />
    <DataGrid x:Name="dg" ItemsSource="{Binding}" />
  </StackPanel>
</Window>
```
```c#
using System.Windows;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Controls;
using System.Linq;

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
      new Dialog(dg).ShowDialog();
    }
  }

  public partial class Dialog : Window
  {
    public Dialog(FrameworkElement fe)
    {
      var b = new Button()
      {
        HorizontalAlignment = HorizontalAlignment.Left,
        VerticalAlignment = VerticalAlignment.Top,
        Content = "test"
      };
      this.Width = 200;
      this.Height = 200;
      b.Click += (s, e) => fe.DataContext = new[] { new { Name = "hello" } };
      this.Content = b;
    }
  }
}
```
