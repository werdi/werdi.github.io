---
title: TreeView SelectedItem's Header in WPF?
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9b9ceecc-33c1-4c1e-bba7-f7b8794f1032/treeview-selecteditems-header-in-wpf?forum=wpf
---
# TreeView SelectedItem's Header in WPF?
*> I cant get the selecteditem's header from treeView? treeView.SelectedItem.ToString(); Not Worked*

if the TreeView is bound to a data source, you can directly get the data source object.
below is an example.
```c#
using System.Collections.Generic;
using System.Windows;
using System.Windows.Controls;

namespace WpfApplication4
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new [] {
        new Node {
        Title = "n1",
        Nodes = new [] { 
          new Node { Title = "n1.1" },
          new Node { Title = "n1.2" }}}};
    }

    private void tv_SelectedItemChanged(object sender, RoutedPropertyChangedEventArgs<object> e)
    {
      var n1 = (e.Source as TreeView).SelectedItem as Node;
      var n2 = e.NewValue as Node;   // n1 == n2
      tb2.Text = "SelectedItem: " + n1.Title + "; e.NewValue: " + n2.Title + ";";
    }
  }

  public class Node
  {
    public string Title { get; set; }
    public IEnumerable<Node> Nodes { get; set; }
  }
}
```
```xaml
<Window x:Class="WpfApplication4.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication4"
  xmlns:sys="clr-namespace:System;assembly=mscorlib"
  Title="MainWindow" Height="350" Width="525" FontSize="16">
  <StackPanel>
    <TreeView x:Name="tv" ItemsSource="{Binding}" SelectedItemChanged="tv_SelectedItemChanged">
      <TreeView.Resources>
        <HierarchicalDataTemplate DataType="{x:Type local:Node}" ItemsSource="{Binding Nodes}">
          <TextBlock Text="{Binding Title}" />
        </HierarchicalDataTemplate>
        <Style TargetType="TreeViewItem">
          <Setter Property="IsExpanded" Value="True" />
        </Style>
      </TreeView.Resources>
    </TreeView>
    <TextBlock Text="{Binding ElementName=tv, Path=SelectedItem.Title}" />
    <TextBlock x:Name="tb2" />
  </StackPanel>
</Window>
```
