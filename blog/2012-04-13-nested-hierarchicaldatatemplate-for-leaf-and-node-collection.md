---
title: Nested HierarchicalDataTemplate for leaf and node collection
tags: [xaml, c#, wpf, template]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a6d3fcf9-724c-4610-87d6-1b952989c37a/nested-hierarchicaldatatemplate-for-leaf-and-node-collection?forum=wpf
---
# Nested HierarchicalDataTemplate for leaf and node collection
*> Nested HierarchicalDataTemplate for leaf and node collection*

try using the following example
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication1"
  Title="MainWindow" Height="350" Width="525">
  <TreeView ItemsSource="{Binding Elements[node]}" ItemTemplate="{DynamicResource node}">
    <TreeView.Resources>
      <Style TargetType="TreeViewItem">
        <Setter Property="IsExpanded" Value="True" />
      </Style>
      <local:NodeTemplateSelector x:Key="nts" />
      <DataTemplate x:Key="leaf">
        <TextBlock Text="{Binding Attribute[name].Value, StringFormat='Leaf: {0}'}" />
      </DataTemplate>
      <HierarchicalDataTemplate ItemsSource="{Binding Elements}" x:Key="node" 
        ItemTemplateSelector="{StaticResource nts}">
        <TextBlock Text="{Binding Attribute[name].Value, StringFormat='Node: {0}'}" />
      </HierarchicalDataTemplate>
    </TreeView.Resources>
  </TreeView>
</Window>
```
```c#
using System.Windows;
using System.Windows.Controls;
using System.Xml.Linq;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      var xe = XElement.Load("..\\..\\XmlFile1.xml");
      this.DataContext = xe;
    }
  }
  public class NodeTemplateSelector : DataTemplateSelector
  {
    public override DataTemplate SelectTemplate(object item, DependencyObject container)
    {
      var xe = item as XElement;
      if (xe == null) return null;
      return (container as FrameworkElement).FindResource(xe.Name.LocalName) as DataTemplate;
    }
  }
}
```
