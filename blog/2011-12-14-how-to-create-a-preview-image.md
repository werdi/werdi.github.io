---
title: WPF Как создать превью картинку
tags: [c#, xaml, wpf, imaging]
src: https://social.msdn.microsoft.com/Forums/ru-RU/6ae2b71d-81b3-4005-ae57-f86316be0152/wpf-?forum=fordesktopru
---
# WPF Как создать превью картинку
* Допустим есть некий ItemsControl к котрому биндится некая коллекция, у него задан шаблон. Как получить, грубо говоря, скриншот данного ItemsControl ?*
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication9"
  Title="MainWindow" Height="350" Width="500" FontSize="16">
  <StackPanel>
    <ListBox ItemsSource="{Binding}" x:Name="lst" />
    <Button Content="Test" Click="Button_Click" HorizontalAlignment="Left" />
    <Image x:Name="img" StretchDirection="DownOnly" />
  </StackPanel>
</Window>
```
```c#
using System.Windows;
using System.Windows.Media;
using System.Windows.Media.Imaging;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new[] { 1, 2, 3 };
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      var bs = Render(lst);
      img.Source = bs;
    }

    BitmapSource Render(FrameworkElement ue)
    {
      var ret = new RenderTargetBitmap((int)ue.ActualWidth, (int)ue.ActualHeight, 96, 96, PixelFormats.Default);
      var dv = new DrawingVisual();
      using (var dc = dv.RenderOpen())
        dc.DrawRectangle(Brushes.White, null, new Rect() { Size = ue.RenderSize });
      ret.Render(dv);
      ret.Render(ue);
      return ret;
    }
  }
}
```
для того, чтобы сохранить BitmapSource, например, в .png-файле:
```c#
var be = new PngBitmapEncoder();
be.Frames.Add(BitmapFrame.Create(bs));   // bs - BitmapSource
using(var f = File.Create("img.png"))
  be.Save(f);
```