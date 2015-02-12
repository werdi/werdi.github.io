---
title: TreeView c CheckBox в WPF
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/ab32a061-cb96-4fed-bbd5-91246f860317/treeview-c-checkbox-wpf?forum=fordesktopru
---
# TreeView c CheckBox в WPF
*> TreeVeiw состоит из чекбоксов, и отображает XML-струкуру. [...] Как идентифицировать эти чекбоксы, чтоб они работали как ветки TreeView, а не как мертвые иконки?*
```xaml
<Window x:Class="WpfApplication8.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication8"
  Title="MainWindow" Height="500" Width="500" FontSize="16">
  <Window.Resources>
    <HierarchicalDataTemplate ItemsSource="{Binding Path=Elements[item]}" x:Key="hdt">
      <WrapPanel>
        <CheckBox IsChecked="{Binding Path=Attribute[value].Value}" Margin="0,5,5,0" />
        <TextBlock Text="{Binding Path=Attribute[name].Value}" />
      </WrapPanel>
    </HierarchicalDataTemplate>
    <Style TargetType="TreeViewItem">
      <Setter Property="IsExpanded" Value="True" />
    </Style>
  </Window.Resources>
  <StackPanel Orientation="Horizontal">
    <TreeView ItemsSource="{Binding Path=Elements[item]}" ItemTemplate="{StaticResource hdt}" />
    <TextBlock Text="{Binding}" x:Name="txt" Margin="10,0,0,0" />
  </StackPanel>
</Window>
```  
```c# 
using System.Windows;
using System.Xml.Linq;

namespace WpfApplication8
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      var xe = XElement.Parse(@"
        <root>
          <item name='i1' value='true'>
            <item name='i11' value='true'>
              <item name='i111' value='true' />    
            </item>
          </item>
        </root>");
      this.DataContext = xe;
      xe.Changed += (s, e) => txt.Text = xe.ToString();
    }
  }
}
```