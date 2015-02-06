---
title: Вопрос по DataGrid
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/1abafff4-6c73-4519-a87a-5bab0ebe37e6/-datagrid?forum=fordesktopru
---
# Вопрос по DataGrid
*> чтобы вводить в такую колонку нужно обязательно дважды кликать.*

чтобы включить редактирование ячейки при ее выделении:
```xaml
<Window x:Class="WpfApplication3.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525">
  <Grid>
    <DataGrid ItemsSource="{Binding}" Name="dg" ColumnWidth="*" />
  </Grid>
</Window>
```
```c#
using System.Data;
using System.Windows;
using System.Windows.Controls;

namespace WpfApplication3
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      var dt = new DataTable();
      dt.Columns.Add("Name", typeof(string));
      for (int i = 0; i < 10; i++)
        dt.Rows.Add("test");

      this.DataContext = dt;

      dg.CurrentCellChanged += (s, e) =>
      {
        var cell = dg.CurrentCell.Column.GetCellContent(dg.CurrentCell.Item).Parent as DataGridCell;
        if (cell.IsEditing == false)
          dg.BeginEdit();
      };
      
      dg.PreviewKeyDown += (s, e) =>
      {
        if(e.Key == System.Windows.Input.Key.Down || e.Key == System.Windows.Input.Key.Up)
          dg.CommitEdit(DataGridEditingUnit.Row, true);
      };
    }
  }
}
```
*> еще один вопрос. На этом датагриде есть такой столбец: [...] DatePicker [...] при переходе в эту ячейку режим редактирования не включается автоматически.*

в примере выше после dg.BeginEdit(); надо добавить следующий код:
```c#
var dp = GetVisualChild<TextBox>(cell);
if(dp != null)
{
  dp.Focus();
  dp.Select(0, 0);
}
  
...
public static T GetVisualChild<T>(Visual parent) where T : Visual
{
  T child = default(T);
  int numVisuals = VisualTreeHelper.GetChildrenCount(parent);
  for (int i = 0; i < numVisuals; i++)
  {
    Visual v = (Visual)VisualTreeHelper.GetChild(parent, i);
    child = v as T;
    if (child == null)
      child = GetVisualChild<T>(v);
    else
      break;
  }
  return child;
}
```
