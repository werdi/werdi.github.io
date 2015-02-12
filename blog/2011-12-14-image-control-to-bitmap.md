---
title: Image Control to Bitmap
tags: [c#, xaml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/73d4ff45-d1aa-457e-8048-c35a79b35c19/image-control-to-bitmap?forum=csharpgeneral
---
# Image Control to Bitmap
*> I am trying to get a System.Drawing.Bitmap out of a System.Windows.Controls.Image. [...] Does anyone know how to do this?*
```c#
var b = new Bitmap(img);
```
see [Bitmap Constructor (Image)](http://msdn.microsoft.com/en-us/library/ts25csc8(v=VS.100).aspx)
*> The constructor for Bitmap accepts a System.Drawing.Image, not a System.Windows.Controls.Image.*

in this case you can use DataObject Class.
below is an example.
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="500" FontSize="16">
  <StackPanel>
    <Button Content="Save" Click="Button_Click" />
    <Image x:Name="img"  />
  </StackPanel>
</Window>
```
```c#
using System;
using System.Windows;
using System.Windows.Media.Imaging;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      img.Source = new BitmapImage(
        new Uri(
         "http://i1.social.s-msft.com/profile/u/avatar.jpg?displayname=malobukv")); 
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      var d = new DataObject(DataFormats.Bitmap, img.Source, true);
      var bmp = d.GetData("System.Drawing.Bitmap") as System.Drawing.Bitmap;
      bmp.Save("res.bmp");
    }
  }
}
```
*> "bmp.Save("res.bmp");" Saving an image as a png with a bmp extension might be consfusing.*

if open the res.bmp in notepad, it starts with "BM...",
but the .png-file starts with "‰PNG...".
