---
title: Control not visible when added to canvas
tags: [wpf, c#, xaml, canvas]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9a9f10cf-469c-48db-a1b5-60060dbb67b5/control-not-visible-when-added-to-canvas?forum=wpf
---
# Control not visible when added to canvas
*> I have a Canvas in wich i place some drawingvisuals. Now i also want to add some buttons and textboxes and more stuff.* 
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="500">
  <Grid x:Name="grd">
    <ItemsControl ItemsSource="{Binding}">
      <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
          <Canvas />
        </ItemsPanelTemplate>
      </ItemsControl.ItemsPanel>
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <TextBox Text="{Binding Title, Mode=OneWay}" />
        </DataTemplate>
      </ItemsControl.ItemTemplate>
      <ItemsControl.ItemContainerStyle>
        <Style>
          <Setter Property="Canvas.Left" Value="{Binding X}" />
          <Setter Property="Canvas.Top" Value="{Binding Y}" />
        </Style>
      </ItemsControl.ItemContainerStyle>
    </ItemsControl>
  </Grid>
</Window>
```
```c#
using System.Windows;

namespace WpfApplication2
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new [] 
      { 
        new { Title = "New button1", X = 10, Y = 100 },
        new { Title = "New button2", X = 70, Y = 50 },
      };
    }
  }
}
```
