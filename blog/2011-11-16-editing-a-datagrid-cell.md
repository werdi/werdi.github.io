---
title: Programmatically edit a datagrid cell in wpf
tags: [c#, xaml, wpf, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/978289bc-f772-4e77-9de5-c85cb6d0c3aa/programmatically-edit-a-datagrid-cell-in-wpf?forum=wpf
---
# Programmatically edit a datagrid cell in wpf
*> I know the rowIndex and the columnIndex. But to travese through the grid is a hard part. I just want something like DataGrid.Rows[0][3].BackgroundColor=WhateverIWant*

instead of traverse through the grid, try using the following method:
```c#
DataGridCell GetCell(DataGrid dg, int rowIndex, int columnIndex)
{
  var dr = dg.ItemContainerGenerator.ContainerFromIndex(rowIndex) as DataGridRow;
  var dc = dg.Columns[columnIndex].GetCellContent(dr);
  return dc.Parent as DataGridCell;
}
```
below is an example
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  FontSize="16" Title="Test" Height="400" Width="500">
  <DataGrid x:Name="dg" ItemsSource="{Binding}" SelectionUnit="Cell" ColumnWidth="*" HeadersVisibility="None" />
</Window> 

```c#
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = Enumerable.Range(0, 10)
        .Select(i => new { Id = i, Text = "text" + i });

      dg.Loaded += new RoutedEventHandler(dg_Loaded);
    }

    void dg_Loaded(object sender, RoutedEventArgs e)
    {
      GetCell(dg, 1, 1).Background = Brushes.Yellow;
    }

    DataGridCell GetCell(DataGrid dg, int rowIndex, int columnIndex)
    {
      var dr = dg.ItemContainerGenerator.ContainerFromIndex(rowIndex) as DataGridRow;
      var dc = dg.Columns[columnIndex].GetCellContent(dr);
      return dc.Parent as DataGridCell;
    }
  }
}
```
you can set the cell's background color by using triggers in xaml. 
below is an example
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  FontSize="16"
  Title="Test" Height="400" Width="500">
  <Grid>
    <Grid.Resources>
      <Style TargetType="DataGridCell">
        <Style.Triggers>
          <!-- select row -->
          <DataTrigger Binding="{Binding Path=Text}" Value="text5">
            <Setter Property="Background" Value="Linen" />
          </DataTrigger>
          <!-- select cell -->
          <MultiDataTrigger>
            <MultiDataTrigger.Conditions>
              <Condition Binding="{Binding Text}" Value="text5" />
              <Condition Binding="{Binding RelativeSource={RelativeSource Self}, Path=Column.DisplayIndex}" Value="1" />
            </MultiDataTrigger.Conditions>
            <Setter Property="Background" Value="Yellow" />
          </MultiDataTrigger>
        </Style.Triggers>
      </Style>
    </Grid.Resources>
    <DataGrid ItemsSource="{Binding}" SelectionUnit="Cell" ColumnWidth="*" HeadersVisibility="None" />
  </Grid>
</Window>
```
```c#
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Linq;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = Enumerable.Range(0, 10)
        .Select(i => new { Id=i, Text = "text"+i });
    }
  }
}
```
*> ... refer to our FAQs: 1.2 How to get the cell content from Datagrid*

the FAQ contains an approach based on DataGridCellsPresenter and GetVisualChild, but there is a shorter one: 
```c#
DataGridCell GetCell(DataGrid dg, int rowIndex, int columnIndex)
{
  var dr = dg.ItemContainerGenerator.ContainerFromIndex(rowIndex) as DataGridRow;
  var dc = dg.Columns[columnIndex].GetCellContent(dr);
  return dc.Parent as DataGridCell;
}
```
I would like to propose to add it to the FAQ. what do you think? 