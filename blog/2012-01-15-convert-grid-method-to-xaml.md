---
title: Convert Grid Method to XAML
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/6bd56a6d-e737-4a7d-8c6f-7eae7a61d747/convert-grid-method-to-xaml?forum=wpf 
---
# Convert Grid Method to XAML
*> In the attached method I create a grid on a canvas using code. Is there a more efficent way to do this using XAML.*
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="500" Width="500" FontSize="16">
  <ItemsControl  ItemsSource="{Binding}">
    <ItemsControl.ItemsPanel>
      <ItemsPanelTemplate>
        <Canvas />
      </ItemsPanelTemplate>
    </ItemsControl.ItemsPanel>
    <ItemsControl.ItemTemplate>
      <DataTemplate>
        <Line X1="{Binding X1}" X2="{Binding X2}" Y1="{Binding Y1}" Y2="{Binding Y2}" 
            Stroke="{Binding Stroke}" StrokeThickness="1" />
      </DataTemplate>
    </ItemsControl.ItemTemplate>
  </ItemsControl>
</Window>
```
```c#
using System.ComponentModel;
using System.Windows;
using System.Windows.Media;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      var list = new System.Collections.Generic.List<object>();
      for (int i = 10; i < 500; i = i + 10)
        list.Add(new { Stroke = Brushes.Blue, X1 = i, X2 = i, Y1 = 0, Y2 = 500 });
      for (int i = 10; i < 500; i = i + 10)
        list.Add(new { Stroke = Brushes.Red, X1 = 0, X2 = 500, Y1 = i, Y2 = i });
      this.DataContext = list;
    }
  }
}
```
