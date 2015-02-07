---
title: Image source
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/7195aee1-3ea0-46bf-89fa-1ec449bd9d4a/image-source?forum=formobiledevicesru 
---
# Image source
*> получаю следующую ошибку: Expected relative Uri, found absolute. [...] new Uri("http://localhost:27926/Content/images/Downloaded/"+ d.Picture ...*

попробуйте убрать хост и порт, т.е. так: new Uri("/Content/images/Downloaded/"+ d.Picture ...

*> Думаю так бы сработало, но вся соль в том что я пытаюсь получить картинку с сайта*

если картинка на чужом сайте, то в Uri надо указать http и UriKind.Absolute
```c#
using System;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media.Imaging;

namespace SilverlightApplication2
{
  public partial class MainPage : UserControl
  {
    public MainPage()
    {
      InitializeComponent();
      img.ImageFailed += (s, e) =>
        System.Diagnostics.Debug.WriteLine(e.ErrorException);
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      Uri uri = new Uri(
        "http://i1.social.s-msft.com/profile/u/avatar.jpg?displayname=malobukv",
        UriKind.Absolute);
      img.Source = new BitmapImage(uri);
    }
  }
}
```
```xaml
<UserControl x:Class="SilverlightApplication1.MainPage"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <Grid>
    <Button HorizontalAlignment="Left" VerticalAlignment="Top" Content="Test" Click="Button_Click" />
    <Image x:Name="img" Margin="0,100,0,0" Height="200" Width="200" />
  </Grid>
</UserControl>
```
