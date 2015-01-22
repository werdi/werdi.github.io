---
title: Получить имя файла при drag'n'drop над WebBrowser'ом
tags: [drag'n'drop, webbrowser, c#]
src: http://mindberg.blogspot.ru/2008/08/dragndrop-webbrowser.html
---
# Получить имя файла при drag'n'drop над WebBrowser'ом
Задача: над WebBrowser'ом сбрасывается файл; надо получить имя файла.
Решение: подписаться на событие Navigating ...
```c#
public partial class Form1 : Form
{
  public Form1()
  {
    WebBrowser wb = new WebBrowser();
    wb.AllowWebBrowserDrop = true;
    wb.Dock = DockStyle.Fill;
    wb.Parent = this;
    wb.Navigating += (s, e) =>
    {
      if (e.Url.Scheme == "file")
      {
        e.Cancel = true;
        MessageBox.Show(e.Url.OriginalString); // путь к файлу
      }
    };
  }
}
```
Примечание: если выполнить drag'n'drop для линка Recycle Bin, то в WebBrowser отобразится содержимое Recycle Bin.
