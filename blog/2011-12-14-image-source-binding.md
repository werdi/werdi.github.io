---
title: Image Source Binding?
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/f153135e-22bf-42e8-9b7d-8153f46d6e61/image-source-binding?forum=fordesktopru
---
# Image Source Binding?
*> Не получается связать Image<Image   Source="{Binding Path=LinearGradient.BitImageSource, Mode=OneWay}"/>с этимpublic class WindowSets : INotifyPropertyChanged ...*
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication9"
  Title="MainWindow" Height="400" Width="280">
  <StackPanel>
    <Button Content="Test" Click="Button_Click" HorizontalAlignment="Left" />
    <Image Stretch="None" Source="{Binding Image}" />
  </StackPanel>
</Window>
```
```c#
using System;
using System.Windows;
using System.ComponentModel;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new Data {
        Image = new Uri("http://i1.social.s-msft.com/profile/u/avatar.jpg?displayname=malobukv")
      };
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      (this.DataContext as Data).Image = 
       new Uri("http://i1.social.s-msft.com/profile/u/avatar.jpg?displayname=a_basic_man");
    }
	}

	public class Data : INotifyPropertyChanged
	{
    public event PropertyChangedEventHandler PropertyChanged = delegate { };
    public Uri Image
    {
      get { return _Image; }
      set
      {
        _Image = value;
        this.PropertyChanged(this, new PropertyChangedEventArgs("Image"));
      }
    }
    Uri _Image;
  }
}
```