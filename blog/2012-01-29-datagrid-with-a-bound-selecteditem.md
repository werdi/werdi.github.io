---
title: Datagrid with a bound SelectedItem doesn't insure the new item is in view when I change it.
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/90bd41f3-f033-4bed-a7c5-b7edb30ccd98/datagrid-with-a-bound-selecteditem-doesnt-insure-the-new-item-is-in-view-when-i-change-it?forum=wpf
---
# Datagrid with a bound SelectedItem doesn't insure the new item is in view when I change it.
*> I have a DataGrid [...] the grid properly selects the new item, but it doesn't insure that the item is visible. Am I missing a setting that's needed to force the visibility or is it handled some other way?*

use DataGrid.ScrollIntoView Method
below is an example.
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="250" FontSize="16">
  <Grid>
    <Button Content="Test" Click="Test_Click" Height="28" VerticalAlignment="Top" />
    <DataGrid ItemsSource="{Binding}" ColumnWidth="*" Margin="0,30,0,0" x:Name="dg" />
  </Grid>
</Window>
```
```c#
using System.Linq;
using System.Windows;
namespace WpfApplication2
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = Enumerable.Range(0, 100).Select(i => new { Key = i });
    }
    private void Test_Click(object sender, RoutedEventArgs e)
    {
      dg.SelectedIndex = -1;
      dg.SelectedIndex = 50;
      dg.ScrollIntoView(dg.SelectedItem);
    }
  }
}
```
