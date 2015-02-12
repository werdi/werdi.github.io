---
title: UserControl width inside a ListBox problem
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/ef0b42dc-5346-4001-9fe4-0ed6a6b6e071/usercontrol-width-inside-a-listbox-problem?forum=wpf
---
# UserControl width inside a ListBox problem
*> I'm having a problem with getting a UserControl items to stretch to the size of its parent, which is a ListBox inside a MainWindow.*

try using the following code
```xaml
<ListBox.Resources>
  <Style TargetType="ListBoxItem">
    <Setter Property="HorizontalContentAlignment" Value="Stretch" />
  </Style>
</ListBox.Resources>
```
also take a look at the [example](http://social.msdn.microsoft.com/Forums/en-SG/wpf/thread/267c9758-d420-4919-b96c-2cf7c13f0ae3/#299b9a20-eda6-4483-aab3-b3c2003e7dda).

*> Try resizing the window on the right side back and forth.*

yes, the right edge of progressbar disappears while decreasing the width of the window.
to solve the problem: set ScrollViewer.HorizontalScrollBarVisibility, ListBoxItem.HorizontalContentAlignment and ListBoxItem.Width;
below is an example. 
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication9"
  Title="MainWindow" Height="400" Width="280" FontSize="16" MinWidth="280">
  <Window.DataContext>
    <CompositeCollection>
      <local:Download
        Image="http://i1.social.s-msft.com/profile/u/avatar.jpg?displayname=malobukv" 
        Name = "Test"
        Progress = "65"
        Size = "450" />
    </CompositeCollection>
  </Window.DataContext>
  <Grid>
    <Image Height="49" HorizontalAlignment="Right" 
        Margin="0,12,12,0" Stretch="Fill" VerticalAlignment="Top" Width="49" />
    <ListBox Margin="0,104,0,32" Name="DownloadList" BorderThickness="0" 
      ItemsSource="{Binding}"
      ScrollViewer.HorizontalScrollBarVisibility="Disabled">
      <ListBox.Resources>
        <Style TargetType="ListBoxItem">
          <Setter Property="HorizontalContentAlignment" Value="Stretch" />
          <Setter Property="Width" Value="{Binding Path=ActualWidth, 
            RelativeSource={RelativeSource Mode=FindAncestor, AncestorType={x:Type Grid}}}" />
        </Style>
      </ListBox.Resources>
      <ListBox.ItemTemplate>
        <DataTemplate>
          <Grid>
            <Grid.ColumnDefinitions>
              <ColumnDefinition Width="53" />
              <ColumnDefinition />
            </Grid.ColumnDefinitions>
            <Image Source="{Binding Path=Image}" Width="49" />
            <StackPanel Grid.Column="1" Margin="0,0,12,0">
              <TextBlock Text="{Binding Path=Name}" Grid.Column="1" />
              <ProgressBar Value="{Binding Path=Progress}" Margin="0,2,0,0" Height="8" Grid.Column="1" />
              <TextBlock Text="{Binding Path=Size}" Margin="0,2,0,0" VerticalAlignment="Top" 
                Grid.Column="1" FontSize="10" />
            </StackPanel>
          </Grid>
        </DataTemplate>
      </ListBox.ItemTemplate>
    </ListBox>
  </Grid>
</Window>
```
```c#
using System;
using System.Windows;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }
  }

  public class Download
  {
    public string Name { get; set; }
    public int Progress { get; set; }
    public Uri Image { get; set; }
    public int Size { get; set; }
  }
}
```