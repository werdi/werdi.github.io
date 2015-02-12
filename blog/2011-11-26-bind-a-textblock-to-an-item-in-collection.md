---
title: How to bind a TextBlock to an individual item from a collection
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/942141fb-bea1-42b6-9fa1-0ed1fc1e4568/how-to-bind-a-textblock-to-an-individual-item-from-a-collection?forum=wpf
---
# How to bind a TextBlock to an individual item from a collection
*> I have an entity, "Person". each "Person" have a collection of "PhoneNumbers" and each "PhoneNumber" have several properties [...] After binding a collection of "Person"s to a DataGrid, I need to bind a specific "PhoneNumber.Number" to a TextBlock in the related DataGrid.RowDetailsTemplate*

if a specific phonenumber mean a specific phonenumber index, then 
below is an example of binding
```xaml
<Window x:Class="WpfApplication7.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication7"
  Title="MainWindow" Height="500" Width="550" FontSize="16">
  <StackPanel>
    <DataGrid AutoGenerateColumns="True" 
      ItemsSource="{Binding}" x:Name="m" IsSynchronizedWithCurrentItem="True">
      <DataGrid.RowDetailsTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding PhoneNumbers[1].Number}" />
        </DataTemplate>
      </DataGrid.RowDetailsTemplate>
    </DataGrid>
    <DataGrid AutoGenerateColumns="True" 
      ItemsSource="{Binding ElementName=m, Path=SelectedItem.PhoneNumbers}" />
  </StackPanel>
</Window>
```
```c#
using System.Collections.Generic;
using System.ComponentModel;
using System.Windows;

namespace WpfApplication7
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = LoadData();
    }

    object LoadData()
    {
      var ret = new BindingList<Person>();
      for (int i = 0; i < 5; i++)
      {
        var ns = new BindingList<PhoneNumber>();
        for (int j = 0; j < 5; j++)
          ns.Add(new PhoneNumber { Number = i + "." + j });
        ret.Add(new Person { Name = "p" + i, PhoneNumbers = ns });
      }
      return ret;
    }
  }

  public class Person
  {
    public string Name { get; set; }
    public ICollection<PhoneNumber> PhoneNumbers { get; set; }
  }

  public class PhoneNumber
  {
    public string Number { get; set; }
  }
 }
```