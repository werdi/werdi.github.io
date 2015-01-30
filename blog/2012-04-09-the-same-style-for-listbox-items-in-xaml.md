---
title: Одинаковый стиль для элементов ListBox'а с заданием значений в файле XAML.
tags: [xaml, c#, wpf, style]
src: https://social.msdn.microsoft.com/Forums/ru-RU/73f0bc45-a978-4cda-ab3c-1ea248afcbb4/-listbox-xaml?forum=formobiledevicesru
---
# Одинаковый стиль для элементов ListBox'а с заданием значений в файле XAML.
*> задать для всех элементов ListBox'а одинаковый стиль, и задание значений элементов ListBox'а делать сразу в файле XAML (не программно в CS). Как это сделать?*

надо определить стиль в ListBox.Resources.

примерно так:
```xaml
<UserControl x:Class="SilverlightApplication1.MainPage"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Width="640" Height="480">
  <ListBox>
    <ListBox.Resources>
      <Style TargetType="ListBoxItem">
        <Setter Property="FontWeight" Value="Bold" />
      </Style>
    </ListBox.Resources>
    <ListBoxItem>1</ListBoxItem>
    <ListBoxItem>2</ListBoxItem>
    <ListBoxItem>3</ListBoxItem>
  </ListBox>
</UserControl>
```
*> как задать ListBoxItem Content для нескольких TextBlock'ов и других элементов.*
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525">
  <ListBox ItemsSource="{Binding}">
    <ListBox.Resources>
      <Style TargetType="ListBoxItem">
        <Setter Property="ContentTemplate">
          <Setter.Value>
            <DataTemplate>
              <Grid>
                <Grid.RowDefinitions>
                  <RowDefinition Height="Auto" />
                  <RowDefinition Height="Auto" />
                </Grid.RowDefinitions>
                <Grid.ColumnDefinitions>
                  <ColumnDefinition Width="Auto" />
                  <ColumnDefinition Width="*" />
                </Grid.ColumnDefinitions>
                <Image Grid.Row="0" Grid.Column="0" Grid.RowSpan="2" Source="{Binding Image}" />
                <TextBlock Grid.Row="0" Grid.Column="1" Text="{Binding Text1}" />
                <TextBlock Grid.Row="1" Grid.Column="1" Text="{Binding Text2}" />
              </Grid>
            </DataTemplate>
          </Setter.Value>
        </Setter>
      </Style>
    </ListBox.Resources>
  </ListBox>
</Window>
```
```c#
using System.Windows;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new[] 
      {
        new {Image = "1.png", Text1 = "text1", Text2 = "text2" },
        new {Image = "2.png", Text1 = "text3", Text2 = "text4" },
      };
    }
  }
}
```
*> А как визуально в дизайне видеть что получилось? Добавить в форму DesignData Source?*

чтобы данные были видны в дизайнере, можно создать отдельный класс.
```c#
public class Model
{
  public Model()
  {
    this.Data = new[] 
    {
      new {Image = "1.png", Text1 = "text1", Text2 = "text2" },
      new {Image = "2.png", Text1 = "text3", Text2 = "text4" },
    };
  }
  public object Data { get; private set; }
}
```
в XAML в тег Window добавить: 
```xaml
xmlns:local="clr-namespace:WpfApplication1"
```
и в ListBox добавить: 
```xaml
<ListBox.DataContext>
  <local:Model/>
</ListBox.DataContext>
```
