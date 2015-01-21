---
title: Как определить префикс у имени XAttribute
tags: [xaml, c#]
src: http://cognitex.blogspot.ru/2009/06/xattribute.html
---
# Как определить префикс у имени XAttribute
Если требуется программно создать xaml:
```xaml
<Grid>
  	<Style x:Key="Style1">
    	<Setter Property="Rectangle.Fill" Value="Yellow" />
  	</Style>
</Grid>
```
то это можно сделать следующим образом: 
```c#
XNamespace ns = "http://schemas.microsoft.com/winfx/2006/xaml";
var grid = new XElement("Grid",
    	new XAttribute(XNamespace.Xmlns + "x", ns),
    	new XElement("Style",
        	new XAttribute(ns.GetName("Key"), "Style1"),
        	new XElement("Setter",
            	new XAttribute("Property", "Rectangle.Fill"),
            	new XAttribute("Value", "Yellow")
            	)
        	)
    	);
```
Определять XAttribute у Grid - обязательно, иначе в теге Style у атрибута Key получим другой префикс:
```xaml
<Style p2:Key="Style1" xmlns:p2="http://schemas.microsoft.com/winfx/2006/xaml">
```
Тоже самое произойдет, если у тега Style вызвать метод Remove().

P.S.
Для того чтобы добавить xmlns="..." в Grid:

```c#
XNamespace xmlns = "http://schemas.microsoft.com/winfx/2006/xaml/presentation";
var grid = new XElement(xmlns.GetName("Grid"), ...
```
