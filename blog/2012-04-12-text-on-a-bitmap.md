---
title: Text on a bitmap
tags: [wpf, c#, xaml, image]
src: https://social.msdn.microsoft.com/Forums/ru-RU/dd98e50a-cc00-4252-bd03-c9b51bf3634d/text-on-a-bitmap?forum=wpf
---
# Text on a bitmap
*> I am trying to put text on an existing bitmap.*

in the WPF you can just overlay text over an image. try using the following code:
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="300" Width="300">
  <Grid>
    <Image Source="http://i1.social.s-msft.com/profile/u/avatar.jpg?displayname=malobukv" Stretch="Fill" />
    <TextBlock Text="Hello" FontSize="60" />
  </Grid>
</Window>
```
*> Can you give an example of your code using the code behind?*
```c#
void Test()
{
  var bmp = new BitmapImage(new Uri("http://i1.social.s-msft.com/profile/u/avatar.jpg?displayname=malobukv"));
  var grd = new Grid();
  grd.Children.Add(new Image { Source = bmp, Stretch = Stretch.Fill });
  grd.Children.Add(new TextBlock { Text = "Hello", FontSize = 60 });
  var wnd = new Window();
  wnd.Content = grd;
  wnd.Show();
}
```
*> I'm just learning WPF, and I'm trying to move the game over from GDI+. I still haven't learned all of the WPF classes, and how they work. It's a lot different than GDI+. *

take a look at the ["An Introduction to Windows Presentation Foundation"] (http://msdn.microsoft.com/en-us/library/aa480192.aspx), especially at "2.3 Retained-mode Graphics"
