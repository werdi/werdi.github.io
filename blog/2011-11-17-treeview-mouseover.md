---
title: Treeview Mouseover
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/3db51101-7b80-41bb-b9bd-9dd7564ab0d8/treeview-mouseover?forum=wpf
---
# Treeview Mouseover
*> Treeview should open the child nodes and its corresponding sub child items upon mouse over.*
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  FontSize="16" Title="Test" Height="400" Width="500">
  <TreeView>
    <TreeView.Resources>
      <Style TargetType="TreeViewItem">
        <Style.Triggers>
          <Trigger Property="IsMouseOver" Value="True">
            <Setter Property="IsExpanded" Value="True"/>
          </Trigger>
          <!-- expand sub nodes-->
          <DataTrigger Binding="{Binding RelativeSource={RelativeSource Mode=Self}, Path=Parent.IsExpanded}" Value="True">
            <Setter Property="IsExpanded" Value="True"/>
          </DataTrigger>
        </Style.Triggers>
      </Style>
    </TreeView.Resources>
    <TreeViewItem Header="n1" >
      <TreeViewItem Header="n2">
        <TreeViewItem Header="n3"/>
        </TreeViewItem>
    </TreeViewItem>
    <TreeViewItem Header="n1" >
      <TreeViewItem Header="n2" />
    </TreeViewItem>
  </TreeView>
</Window>
```
```c#
using System.Windows;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }
  }
}
```