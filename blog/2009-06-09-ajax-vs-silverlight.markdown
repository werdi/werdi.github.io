---
title: AJAX vs Silverlight
tags: [ajax, silverlight, xaml]
src: http://mindberg.blogspot.ru/2008/06/ajax-vs.html
---
# AJAX vs Silverlight
Вообще-то AJAX нельзя сравнивать с Silverlight'ом, потому что AJAX предназначен ТОЛЬКО для обмена данными между браузером и сервером. А Silverlight - это графика, видео, анимация, обмен данными, реализация логики на современном языке программирования C# и многое другое в одном флаконе (бесплатно, весит 4Mb). Silverlight создан в Microsoft в рамках стратегии Web user experience (UX).

Предположим, что есть следующая Задача: надо в браузере отобразить прямоугольник (заливка: вертикальный градиент - зеленый в синий, размер: 800 px на 3000 px) с текстом Hello (цвет: белый); при клике на Hello надо отправить на сервер, например, время клиента. Решение: открыть графический редактор, создать в нем прямоугольник с градиентной заливкой, написать слово Hello; сохранить в файле в одном из форматов (jpeg, gif, png); создать html-страницу, добавить в нее тег img ... и т.д. и т.п; да, еще надо написать на Jscript код, который обработает нажатие (с этим придется повозиться). Минусы: вес у jpg-файла получится больше 40Kb; графику создавалась вручную.

Но так было раньше. А теперь Решение (на Silverlight 2): создается xaml-файл, примерно такой:
```xaml
<UserControl x:Class="SilverlightApplication2.Page"
xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" Width="800" Height="3000">
  <Canvas>
    <Rectangle Stroke="#FF000000" Width="800" Height="3000">
      <Rectangle.Fill>
        <LinearGradientBrush>
          <LinearGradientBrush.GradientStops>
            <GradientStop Color="#FF00FF00" Offset="0"/>
            <GradientStop Color="#FF0000FF" Offset="1"/>
          </LinearGradientBrush.GradientStops>
          </LinearGradientBrush>
      </Rectangle.Fill>
    </Rectangle>
    <TextBlock Text="Hello" FontFamily="Georgia" FontSize="18" Margin="100,10,10,10"/>
  </Canvas>
</UserControl>
```
И все! Никаких графических редакторов не надо. И самое главное, что "Hello" можно заменить программно. Обработчик клика можно писать не на Jscript, а на C# и не надо никакой возни с определением положения текста в рисунке! Анимация тоже делается достаточно просто. После того как xaml-файл создан, он загружается в Silverlight (ActiveX-компонент). Кстати, Silverlight существует для разных браузеров (под Windows, Linux, КПК и смартфонов) и работает везде одинаково.

Существует удобный графический редактор для создания xaml-файлов - это Expression Design, который входит в состав Microsoft Expression Studio. Кстати, многие из тех, кто был на remix.ru получили Expression Studio в подарок, так что не пропустите следующий remix.

P.S.
некоторые называют Silverlight - сервелат :)
