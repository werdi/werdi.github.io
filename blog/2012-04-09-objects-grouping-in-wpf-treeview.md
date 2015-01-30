---
title: WPF. Группировка объектов в TreeView
tags: [wpf, xaml, c#, treeview]
src: https://social.msdn.microsoft.com/Forums/ru-RU/fdc3b0a2-6a41-4c24-9313-bb5dc8ad2dd1/wpf-treeview?forum=fordesktopru
---
# WPF. Группировка объектов в TreeView
*> вопрос: как сделать то же самое, только без построения нового дерева? То есть средствами XAML достать нужные данные из начальной структуры*

в xaml нет поддержки xpath, поэтому надо создать конвертер и явно указать иерархию шаблонов. 

примерно так:
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="500"
  xmlns:local="clr-namespace:WpfApplication2">
  <Grid>
    <Grid.Resources>
      <Style TargetType="TreeViewItem">
        <Setter Property="IsExpanded" Value="True" />
      </Style>
      <DataTemplate x:Key="ot">
        <TextBlock Text="Object" />
      </DataTemplate>
      <HierarchicalDataTemplate x:Key="lt" ItemsSource="{Binding Converter={local:Converter}}" 
        ItemTemplate="{StaticResource ot}">
        <TextBlock Text="Layer" />
      </HierarchicalDataTemplate>
      <HierarchicalDataTemplate x:Key="st" ItemsSource="{Binding Subsystems}" 
        ItemTemplate="{StaticResource lt}">
        <TextBlock Text="System" />
      </HierarchicalDataTemplate>
    </Grid.Resources>
    <TreeView ItemsSource="{Binding}" ItemTemplate="{StaticResource st}" />
  </Grid>
</Window>
```
```c#
using System;
using System.Collections.Generic;
using System.Globalization;
using System.Windows;
using System.Windows.Data;
using System.Windows.Markup;
namespace WpfApplication2
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      var sys = new MSystem()
      {
        Subsystems = new List<MSubsystem>()
        {
          new MSubsystem() { Object = new MObject() { Layer = new MLayer() }},
          new MSubsystem() { Object = new MObject() { Layer = new MLayer() }},
        }
      };
      this.DataContext = new[] { sys };
    }
  }
  public class Converter : MarkupExtension, IValueConverter
  {
    public Converter() { }
    public override object ProvideValue(System.IServiceProvider serviceProvider) { return this; }
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
      var s = value as MSubsystem;   // далее находим подэлементы, группируем, ...
      return new[] { new MObject(), new MObject() };     // test data
    }
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
      throw new System.NotImplementedException();
    }
  }
  public class MSystem { public List<MSubsystem> Subsystems { get; set; } }
  public class MSubsystem { public MObject Object { get; set; } }
  public class MObject { public MLayer Layer { get; set; } }
  public class MLayer { }
}
```
p.s.
другой способ изменения treeview см. [здесь] (http://social.msdn.microsoft.com/Forums/ru/fordesktopru/thread/028687df-cfe1-421f-89f5-33342e8dc8f7).

*> не хочу создавать новые классы. Хочу привязаться к уже существующей структуре. Чтобы при изменении коллекции, привязанной к TreeView, не нужно было насильно обновлять этот элемент.* 

если классы GraphLayer, Subsystem и GraphObject не реализуют интерфейс INotifyPropertyChanged, то  динамического обновления UI при изменении данных не будет.
