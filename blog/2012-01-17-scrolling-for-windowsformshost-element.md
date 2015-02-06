---
title: Скроллинг для элемента из WindowsFormsHost
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/1029dedf-1a60-4d91-a6cc-65d43a63f0b9/-windowsformshost?forum=fordesktopru
---
# Скроллинг для элемента из WindowsFormsHost
*> необходимо, чтобы при нажатии на кнопку button1 в pictureBox1 загружалось изображение*

в WPF для вывода изображения есть Image:
```xaml
<Window x:Class="WpfApplication3.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525">
  <Grid>
    <Button HorizontalAlignment="Left" VerticalAlignment="Top" Click="Button_Click" Content="Test" />
    <Image x:Name="img" Margin="0,30,0,0" />
  </Grid>
</Window>
```
```c#
using System;
using System.Windows;
using System.Windows.Media.Imaging;

namespace WpfApplication3
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      img.Source = new BitmapImage(new Uri(@"D:\Temp\test.png"));
    }
  }
}
```
