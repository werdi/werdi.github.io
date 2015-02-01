---
title: Как найти CheckBox на WPF форме по его DisplayName
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b0a7a28c-ac26-432e-a721-b9bd85313aa1/-checkbox-wpf-displayname?forum=fordesktopru 
---
# Как найти CheckBox на WPF форме по его DisplayName
*> Как найти CheckBox на WPF форме*

обычно искать не требуется. просто в xaml для CheckBox указываете x:Name, напимер:
```xaml
<CheckBox x:Name="cb1" />
```
в коде пишете, например:
```c#
cb1.IsChecked = true;
```
*> CheckBox в xaml не прописан.. он добавляется на форму позже.. изначально форма пустая..*
в такой ситуации можно использовать XamlReader.Parse и FrameworkElement.FindName.
примерно так:
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="500" Width="500">
  <StackPanel>
    <Button Content="dialog" HorizontalAlignment="Left" VerticalAlignment="Top" Click="Button_Click" />
    <StackPanel x:Name="sp"></StackPanel>
  </StackPanel>
</Window>
```
```c#
using System.Windows;
using System.Windows.Controls;
using System.Windows.Markup;
using System.Windows.Media;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      var pc = new ParserContext();
      pc.XmlnsDictionary.Add("", "http://schemas.microsoft.com/winfx/2006/xaml/presentation");
      pc.XmlnsDictionary.Add("x", "http://schemas.microsoft.com/winfx/2006/xaml");
      var o = (StackPanel) XamlReader.Parse(@"
        <StackPanel>
          <TextBox /> 
          <CheckBox x:Name='cb1' />
        </StackPanel>", 
        pc);
      var c = (CheckBox) o.FindName("cb1");
      c.Background = Brushes.Red;
      sp.Children.Add(o);
    }
  }
}
```
