---
title: Работа с ListBox в WP7
tags: [c#, xaml, silverlight]
src: https://social.msdn.microsoft.com/Forums/ru-RU/58b11fa3-cbbb-4d97-ac6b-75e77d64605c/-listbox-wp7?forum=formobiledevicesru
---
# Работа с ListBox в WP7
*> требуется точно позиционировать выбранный элемент по верхней границе видимой области то есть чтобы он был первым.*
```xaml
<UserControl x:Class="SilverlightApplication5.MainPage"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <StackPanel HorizontalAlignment="Left" Width="300" Margin="10">
    <Button Content="Test" Click="Test_Click" 
        VerticalAlignment="Top" HorizontalAlignment="Left" />
    <ListBox ItemsSource="{Binding}" Height="200" x:Name="lb" />
  </StackPanel>
</UserControl>
```
```c#
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;

namespace SilverlightApplication5
{
  public partial class MainPage : UserControl
  {
    public MainPage()
    {
      InitializeComponent();
      this.DataContext = Enumerable.Range(0, 100).Select(i => new { Key = i });
    }
    void Test_Click(object sender, RoutedEventArgs e)
    {
      lb.SelectedItem = lb.Items[50];
      var sv = FindChild<ScrollViewer>(lb);
      if(sv != null) sv.ScrollToVerticalOffset(lb.SelectedIndex);
    }
    T FindChild<T>(DependencyObject o) where T : DependencyObject
    {
      if (o is T) 
        return (T) o;
      for (int i = 0; i < VisualTreeHelper.GetChildrenCount(o); i++)
      {
        var child = VisualTreeHelper.GetChild(o, i);
        var result = FindChild<T>(child);
        if (result != null) return (T) result;
      }
      return null;
    }
  }
}
```
