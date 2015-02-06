---
title: WPF DataGrid Binding
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/634fa014-6e7a-4e96-93b2-4c040e42bb0f/wpf-datagrid-binding?forum=wpf
---
# WPF DataGrid Binding
*> I have a custom object that has two string properties, Make & Model. It also contains a custom List [...] the user can click a row and expand that data into another datagrid. I want that datagrid to be bound to the items within the CustomList*
```xaml
<Window x:Class="WpfApplication3.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="420" Width="525" SizeToContent="Height">
  <StackPanel>
    <DataGrid ItemsSource="{Binding}" Name="dg" ColumnWidth="*" Height="200"
        AutoGenerateColumns="False" VerticalGridLinesBrush="Silver" 
        IsSynchronizedWithCurrentItem="True">
      <DataGrid.Columns>
        <DataGridTextColumn Header="Make" Binding="{Binding Make}" />
        <DataGridTextColumn Header="Model" Binding="{Binding Model}" />
      </DataGrid.Columns>
    </DataGrid>
    <DataGrid ItemsSource="{Binding ElementName=dg, Path=SelectedItem.List}" ColumnWidth="*"  
          Height="200" VerticalGridLinesBrush="Silver" />
  </StackPanel>
</Window>
```
```c#
using System.Data;
using System.Windows;
using System.Windows.Controls;
using System.Collections.Generic;

namespace WpfApplication3
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      this.DataContext = new [] 
      {
        new Obj { Make = "m1", Model="m1", 
          List = new [] { new {Name="c1"}, new {Name="c2"}} },
        new Obj { Make = "m2", Model="m2", 
          List = new [] { new {Name="c3"}, new {Name="c4"}} }
      };
    }
  }

  class Obj
  {
    public string Make { get; set; }
    public string Model { get; set; }
    public IEnumerable<object> List { get; set; }
  }
}
```
