---
title: Аналог ASP/NET Repeater в Silverlight
tags: [c#, xaml, silverlight]
src: https://social.msdn.microsoft.com/Forums/ru-RU/db661ef1-9fe7-4fcb-9079-9bd8f4c3b414/-aspnet-repeater-silverlight?forum=aspnetru
---
# Аналог ASP/NET Repeater в Silverlight
*> нужен какой то контрол, аналог Repeater в ASP.NET. Есть ли что нибудь подобное в Silverlight? [...]  мне нужно вывести кнопки горизонтально*

используйте ItemsControl, с переопределенным ItemsPanel и ItemTemplate
```xaml
<UserControl
	xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
	x:Class="SilverlightApplication3.MainPage"
	Width="640" Height="480">
  <ItemsControl ItemsSource="{Binding}">
    <ItemsControl.ItemsPanel>
      <ItemsPanelTemplate>
        <StackPanel Orientation="Horizontal" />
      </ItemsPanelTemplate>      
  	</ItemsControl.ItemsPanel>
  	<ItemsControl.ItemTemplate>
      <DataTemplate>
        <Button Content="{Binding}" />
      </DataTemplate>
    </ItemsControl.ItemTemplate>
  </ItemsControl>
</UserControl>
```
```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Animation;
using System.Windows.Shapes;

namespace SilverlightApplication3
{
	public partial class MainPage : UserControl
	{
		public MainPage()
		{
			InitializeComponent();
			this.DataContext = new [] { "hello", "world"};
		}
	}
}
```
