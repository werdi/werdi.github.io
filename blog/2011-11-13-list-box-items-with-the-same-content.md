---
title: List box items with the same content
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/2343ed95-5267-4e9d-96f7-1d3655397a5a/list-box-list-box-items-with-same-same-content?forum=wpf
---
# List box items with the same content
*> if i select berry in 9th some times all the berrys are getting selected mostly next to it.*

try to use SelectedValuePath.
below is an example
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525">
  <Grid>
    <ListBox x:Name="lb" ItemsSource="{Binding}" 
      DisplayMemberPath="Name" SelectedValuePath="Id" />
  </Grid>
</Window>
```
```c#
using System;
using System.Collections;
using System.Linq;
using System.Windows;
using System.Windows.Input;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = CreateList();
    }

    object CreateList()
    {
      var str = @"
      Apple    <---- 1
      Berry    <---- 2
      Berry    <---- 3
      Orange    <---- 4
      Apple    <---- 5
      Apple    <---- 6
      Grape    <---- 7
      Apple    <---- 8
      Berry    <---- 9
      Berry    <---- 10
      Grape    <---- 11
      Grape    <---- 12";
      var arr = str.Split(new[] { '\n', '\r' }, StringSplitOptions.RemoveEmptyEntries);
      return from line in arr
        let res = line.Split(new string[] { "<----" }, StringSplitOptions.RemoveEmptyEntries)
        select new { Name = res[0].Trim(), Id = res[1].Trim() };
    }
  }
}
```