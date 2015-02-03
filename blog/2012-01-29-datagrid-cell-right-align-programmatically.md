---
title: Datagrid cell right align programmatically
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a15f49d9-a866-46f9-be01-cf0ed9ff8bc3/datagrid-cell-right-align-programmatically?forum=wpf 
---
# Datagrid cell right align programmatically
*> how to right align my datagridtext columns programmatically*
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="250" FontSize="16">
  <Window.Resources>
    <Style TargetType="DataGridCell">
      <Setter Property="HorizontalAlignment" Value="Right" />
    </Style>
  </Window.Resources>
  <StackPanel>
    <DataGrid ItemsSource="{Binding}" ColumnWidth="*" x:Name="dg" />
    <DockPanel HorizontalAlignment="Left">
      <Button Content="Left" Click="Left_Click" />
      <Button Content="Right" Click="Right_Click" />
    </DockPanel>
  </StackPanel>
</Window>
```
```c#
using System.Windows;
using System;
using System.Windows.Controls;
namespace WpfApplication2
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new[] { new { Id = 1 }, new { Id = 2 } };
    }
    private void Left_Click(object sender, RoutedEventArgs e)
    {
      Align(dg.Columns[0], TextAlignment.Left);
    }
    private void Right_Click(object sender, RoutedEventArgs e)
    {
      Align(dg.Columns[0], TextAlignment.Right);
    }

    void Align(DataGridColumn c, TextAlignment a)
    {
      Style s = new Style();
      s.Setters.Add(new Setter(TextBlock.TextAlignmentProperty, a));
      c.CellStyle = s;
    }
  }
}
```
