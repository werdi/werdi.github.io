---
title: WPF. Отступы для первого и последнего элементов списка
tags: [c#, xaml, wpf, datagrid]
src: https://social.msdn.microsoft.com/Forums/ru-RU/09ccbd14-4a4a-476e-87d3-6a10e951c8dd/wpf-?forum=fordesktopru
---
# WPF. Отступы для первого и последнего элементов списка
*> Есть ItemsControl. Необходимо для первого и последнего элементов сделать отличные от других элементов Margin-ы.*
```xaml
<Window x:Class="WpfApplication3.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" FontSize="16" Width="300" Height="400" 
  xmlns:app="clr-namespace:WpfApplication3">
  <StackPanel>
    <ItemsControl ItemsSource="{Binding}">
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding Id}">
            <TextBlock.Margin>
              <MultiBinding Converter="{app:MarginConverter}">
                <Binding RelativeSource="{RelativeSource AncestorType=ItemsControl}" Path="Items" />
                <Binding RelativeSource="{RelativeSource Self}" Path="DataContext" />
              </MultiBinding>
            </TextBlock.Margin>
          </TextBlock>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
    </StackPanel>
</Window>
```
```c#
using System;
using System.Globalization;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Markup;

namespace WpfApplication3
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext =
        from x in Enumerable.Range(0, 5) select new { Id = x, Value = "v" + x, Note = "n" + x };
    }
  }

  class MarginConverter : MarkupExtension, IMultiValueConverter
  {
      public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
      var ic = values[0] as ItemCollection;
      var dc = values[1];

      var ret = new Thickness();
      var i = ic.IndexOf(dc);
      if (i == 0 || i == ic.Count - 1) ret.Left = 20;

      return ret;
    }
    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
      throw new NotImplementedException();
    }
    public override object ProvideValue(IServiceProvider serviceProvider) { return this; }
  }
}
```
а для DataGrid так:
```xaml
<Window x:Class="WpfApplication3.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" FontSize="16" Width="300" Height="400" 
  xmlns:app="clr-namespace:WpfApplication3">
  <StackPanel>
    <DataGrid ItemsSource="{Binding}" ColumnWidth="*" HeadersVisibility="Column">
      <DataGrid.CellStyle>
        <Style TargetType="DataGridCell">
          <Setter Property="Margin">
            <Setter.Value>
              <MultiBinding Converter="{app:CellMarginConverter}" ConverterParameter="0">
                <Binding RelativeSource="{RelativeSource Self}" />
                <Binding RelativeSource="{RelativeSource AncestorType=DataGridRow}" />
                <Binding RelativeSource="{RelativeSource AncestorType=DataGrid}" />
              </MultiBinding>
            </Setter.Value>
          </Setter>
        </Style>
      </DataGrid.CellStyle>
    </DataGrid>
  </StackPanel>
</Window>
```
```c#
using System;
using System.Globalization;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Markup;

namespace WpfApplication3
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext =
        from x in Enumerable.Range(0, 5) select new { Id = x, Value = "v" + x, Note = "n" + x };
    }
  }

  class CellMarginConverter : MarkupExtension, IMultiValueConverter
  {
    public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
      var c = values[0] as DataGridCell;
      var r = values[1] as DataGridRow;
      var d = values[2] as DataGrid;
      var ret = r.Margin;
      if (c.Column.DisplayIndex.ToString() == parameter.ToString())
      {
        var ri = r.GetIndex();
        if (ri == 0 || ri == d.Items.Count - 1)
          ret.Left += 20;
      }
      return ret;
    }
    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
      throw new NotImplementedException();
    }
    public override object ProvideValue(IServiceProvider serviceProvider) { return this; }
  }
}
```
