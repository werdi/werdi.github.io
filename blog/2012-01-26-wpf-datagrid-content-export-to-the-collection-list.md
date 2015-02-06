---
title: Экспорт содержимого wpf DataGrid в коллекцию List(Of int)
tags: [c#, xaml, wpf, linq, reflection]
src: https://social.msdn.microsoft.com/Forums/ru-RU/ad790d79-75c7-4753-abee-10f09ef077be/-wpf-datagrid-listint?forum=fordataru
---
# Экспорт содержимого wpf DataGrid в коллекцию List(Of int)
*> Мне нужно получить весь DataGrid [...] разместить построчно в двумерной коллекции*

получить элементы можно через свойство Items
и для каждого элемента прочесть значения свойств.
см. ниже метод ReadAll(IEnumerable src)
```xaml
<Window x:Class="WpfApplication3.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Width="500" FontSize="16" SizeToContent="Height" >
  <StackPanel>
    <DockPanel HorizontalAlignment="Left">
      <Button Content="Test1" Click="Test1_Click" />
      <Button Content="Test2" Click="Test2_Click" />
    </DockPanel>
    <DataGrid x:Name="dg1" ItemsSource="{Binding Test1}" />
    <DataGrid x:Name="dg2" ItemsSource="{Binding Test2}" 
        CanUserAddRows="False" />
    <TextBox x:Name="rtb" TextWrapping="Wrap" Height="200"/>
  </StackPanel>
</Window>
```
```c#
using System.Collections;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Linq;
using System.Reflection;
using System.Text;
using System.Windows;

namespace WpfApplication3
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      // test1
      var data1 = new[] 
      { 
        new { Name = 11, Value = 12 },
        new { Name = 21, Value = 22 }
      };

      // test2
      var data2 = new DataTable();
      data2.Columns.Add("Id", typeof(int));
      data2.Columns.Add("Value", typeof(string));
      for (int i = 0; i < 5; i++)
        data2.Rows.Add(i, "value" + i);

      this.DataContext = new { Test1 = data1, Test2 = data2 };
    }
    private void Test1_Click(object sender, RoutedEventArgs e)
    {
      Trace(dg1.Items);
    }
    private void Test2_Click(object sender, RoutedEventArgs e)
    {
      Trace(dg2.Items);
    }
    private void Trace(IEnumerable src)
    {
      var sb = new StringBuilder();
      foreach(var line in ReadAll(src))
      {
        foreach(var itm in line) sb.Append(itm).Append("; ");
        sb.AppendLine();
      }
      rtb.Text = sb.ToString();
    }
    IEnumerable<IEnumerable<object>> ReadAll(IEnumerable src)
    {
      foreach (var itm in src)
      {
        var ct = itm as ICustomTypeDescriptor;
        if (ct != null)
        {
          yield return from x in ct.GetProperties().OfType<PropertyDescriptor>()
                        select x.GetValue(ct);
        }
        else
        {
          yield return from x in itm.GetType().GetProperties().OfType<PropertyInfo>()
                        select x.GetValue(itm, new object[0]);
        }
      }
    }
  }
}
```
