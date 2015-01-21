---
title: XAML-> PNG | GIF | JPG | ...
tags: [drawing, xaml, c#, png, gif,jpg]
src: http://cognitex.blogspot.ru/2009/07/xaml-png-gif-jpg.html
---
# XAML-> PNG | GIF | JPG | ... 
Из XAML можно получить изображение в формате PNG, GIF, JPG. Ниже пример создания PNG-файла.
```c#
using System.IO;
using System.Windows;
using System.Windows.Media;
using System.Windows.Media.Imaging;

public class XamlHelper
{
  public static void SaveToPNG(FrameworkElement fe, string fname)
  {
    var bitmap = new RenderTargetBitmap((int)fe.ActualWidth, (int)fe.ActualHeight, 96, 96, PixelFormats.Pbgra32);
    bitmap.Render(fe);
    BitmapEncoder encoder = new PngBitmapEncoder();
    encoder.Frames.Add(BitmapFrame.Create(bitmap));
    using (Stream stream = File.Create(fname))
    encoder.Save(stream);
  }
}
```
Также можно создать: GIF - с помощью GifBitmapEncoder, JPG - JpegBitmapEncoder.

P.S.
код можно использовать в WinForms приложении или обычной Class Library. Для этого к проекту необходимо подключить следующие сборки:
C:\Program Files\Reference Assemblies\Microsoft\Framework\v3.0\WindowsBase.dll
C:\Program Files\Reference Assemblies\Microsoft\Framework\v3.0\PresentationFramework.dll
C:\Program Files\Reference Assemblies\Microsoft\Framework\v3.0\PresentationCore.dll 
