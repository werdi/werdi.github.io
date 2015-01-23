---
title: Использование Silverlight
tags: [silverlight, html]
src: http://mindberg.blogspot.ru/2008/06/silverlight.html
---
# Использование Silverlight
Вывод графики на веб-страницу можно организовать с помощью Silverlight. Например, надо вывести черный квадрат. Ниже приводится код страницы с встроенной графикой:
```html
<html>
  <body>
    <script type="'text/xaml'" id="'scene'"><?xml version='1.0'?>
      <canvas xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" x="http://schemas.microsoft.com/winfx/2006/xaml">
        <rectangle width="50" height="50" stretch="Fill" fill="Black">
      </canvas>
    </script>
    <object type="application/ag-plugin" classid="clsid:DFEAF541-F3E1-4C24-ACAC-99C30715084A"
      style="border: 0px; width: 100px; height: 100px;">
      <param name="source" value="#scene">
    </object>
  </body>
</html>
```
Если установлен Silverlight (версия 2.0 - весит чуть больше 4Mb), то можно код примера сохранить в текстовом файле с расширением .htm и открыть в IE.

Усложним задачу. Добавим текст Hello и при щелчке левой кнопкой мыши будем выводить сообщение. Ниже в примере болдом выделено то, что было добавлено:
```html
<html>
  <body>
    <script>
      function showDlg()
      {
        alert(1);
      }
    </script>
    <script type='text/xaml' id='scene'><?xml version='1.0'?>
      <Canvas xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
        <Rectangle Width="50" Height="50" Stretch="Fill" Fill="Red" />
        <TextBlock Text="Hello" MouseLeftButtonDown="showDlg" />
      </Canvas>
    </script>
    <object type="application/ag-plugin" classid="clsid:DFEAF541-F3E1-4C24-ACAC-99C30715084A"
      style="border: 0px; width: 100px; height: 100px;">
      <param name="source" value="#scene" />
    </object>
  </body>
</html>
```
P.S.
Кстати, в Silverlight есть множество контролов, которые поддерживают связывание данных и т.д. А обработчики можно создавать на C# в бесплатной Visual Studio. 
Так что же выбрать для реализации интранет сайта: Silverlight или флеш (Adobe Flash)? ;)
