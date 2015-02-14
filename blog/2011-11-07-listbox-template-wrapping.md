---
title: ListBox Template Wrapping question
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/267c9758-d420-4919-b96c-2cf7c13f0ae3/listbox-template-wrapping-question?forum=wpf
---
# ListBox Template Wrapping question
*> ... 2 questions [...] the ItemTemplete does not stretch to the size of the list box [...]  the tweet being cut off.*

you've to set necessary values for ListBox HorizontalScrollBarVisibility, ListBoxItem HorizontalContentAlignment, and TextBlock TextWrapping. 
below is an example.
```xaml
<Window x:Class="WpfApplication0.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication0"
  xmlns:sys="clr-namespace:System;assembly=mscorlib"
  FontSize="16"
  Title="MainWindow" Width="400" Height="600">
  <Grid x:Name="g">
    <ListBox x:Name="lb" ItemsSource="{Binding}"
      ScrollViewer.HorizontalScrollBarVisibility="Disabled">
      <ListBox.Resources>
        <Style TargetType="ListBoxItem">
          <Setter Property="HorizontalContentAlignment" Value="Stretch" />
        </Style>
      </ListBox.Resources>
      <ListBox.ItemTemplate>
        <DataTemplate>
          <Grid Margin="0,10,0,0">
            <Grid.ColumnDefinitions>
              <ColumnDefinition Width="60" />
              <ColumnDefinition />
            </Grid.ColumnDefinitions>
            <Image Source="{Binding Image}" Width="50" Height="50" Grid.Column="0" />
            <WrapPanel Orientation="Vertical" Grid.Column="1">
              <TextBlock Text="{Binding Name, StringFormat='@{0}'}" TextDecorations="Underline" />
              <TextBlock Text="{Binding Text}" Width="Auto" TextWrapping="Wrap"  />
            </WrapPanel>
          </Grid>
        </DataTemplate>
      </ListBox.ItemTemplate>
    </ListBox>
  </Grid>
</Window>
```
```c# 
using System.Windows;
using System.Linq;
using System;

namespace WpfApplication0
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
      var rnd = new Random();  // for test
      return Enumerable.Range(0, 10).Select(i => new 
      {
        Name = "name"+i,
        Image = "",
        Text = String.Concat(Enumerable.Repeat("text ", rnd.Next(10, 30))) + "."
      });
    }
  }
}
```