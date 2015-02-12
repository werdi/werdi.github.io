---
title: Работа с DataGrid в Silverlight
tags: [c#, xaml, wpf, silverlight]
src: https://social.msdn.microsoft.com/Forums/ru-RU/986d2003-b8a5-4252-bcdd-15bfc1fc5401/-datagrid-silverlight?forum=aspnetru
---
# Работа с DataGrid в Silverlight
*> при выборе строки (кликаем мышкой), считывать данные из определённой ячейки выделенной строки в textbox. Перерыл много статей в интернете но работающего решения так и не нашёл.*
если DataGrid привязан к данным (например, коллекция объектов; у объекта есть свойство Prop1), то TextBox.Text можно привязать к текущему элементу в DataGrid.
примерно так делается в wpf. скорее всего, что в silverlight также:
```xaml
<DataGrid ItemsSource="{Binding}" IsSynchronizedWithCurrentItem="True" x:Name="dg" />
<TextBox Text="{Binding ElementName=dg, Path=SelectedItem.Prop1}" />
```
*DataGrid заполняется данными полученых из сервиса, свойство Prop1 не нашёл.*

данные - это коллекция объектов. у объектов есть свойства. вместо Prop1 надо указать название свойства объекта.
```xaml
<UserControl
	xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
	xmlns:sdk="http://schemas.microsoft.com/winfx/2006/xaml/presentation/sdk"
	xmlns:local="clr-namespace:SilverlightApplication1"
	x:Class="SilverlightApplication1.MainPage"
	Width="640" Height="480">

	<StackPanel>
		<sdk:DataGrid  Margin="10" AutoGenerateColumns="True" x:Name="dg" 
			ItemsSource="{Binding Data}" RowHeight="25" SelectedIndex="0" >
			<sdk:DataGrid.DataContext>
				<local:Model/>
			</sdk:DataGrid.DataContext>
		</sdk:DataGrid>	
		<TextBlock Text="{Binding Path=SelectedItem.Data, ElementName=dg}" />				
	</StackPanel>
</UserControl>
```
```c#
using System;
using System.Collections.Generic;
using System.Windows;
using System.Windows.Controls;

namespace SilverlightApplication1
{
	public partial class MainPage : UserControl
	{
		public MainPage()
		{
			InitializeComponent();
		}
	}
	public class Model
	{
		public Model()
		{
			this.Data = new [] 
			{
				new { Id=1, Data="d1" },
				new { Id=2, Data="d2" },
			};
		}
		public object Data { get; private set; }
	}
}
```