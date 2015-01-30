---
title: Передача данных в другой UserControl
tags: [c#, xaml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/dd25035a-b1e2-4034-bef4-dad15f132e69/-usercontrol?forum=fordesktopru
---
# Передача данных в другой UserControl
*> А как мне загрузить данные в TextBlock находящийся во втором UserControl?*
если используете привязку данных, то второму UserControl надо передать DataContext.
и указать привязку. например, TextBlock.Text выводит значение свойства MyProp: 
```xaml
<TextBlock Text="{Binding MyProp}" />
```
*> А без привязки?*
если значение меняется из cs-кода, то надо указать имя_контрола.textBox5.Text

[MainWindow.xaml.cs]
```c#
using System.Windows;
namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      u1.t1.Text = "text1";
      u2.textBox5.Text = "text2";
    }
  }
}
```
[MainWindow.xaml]
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525"
  xmlns:user="clr-namespace:WpfApplication1">
  <StackPanel>
    <user:UserControl1 x:Name="u1" />
    <user:UserControl2 x:Name="u2" />
  </StackPanel>
</Window>
```
[UserControl1.xaml]
```xaml
<UserControl x:Class="WpfApplication1.UserControl1"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <TextBlock x:Name="t1" />
</UserControl>
```
[UserControl2.xaml]
```xaml
<UserControl x:Class="WpfApplication1.UserControl2"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <TextBlock x:Name="textBox5"/>
</UserControl>
```
p.s.
но лучше использовать привязку данных. примерно так:

[MainWindow.xaml.cs]
```c#
using System.Windows;
namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new { Text1 = "text1", Text2 = "text2" };
    }
  }
}
```
[MainWindow.xaml]
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525"
  xmlns:user="clr-namespace:WpfApplication1">
  <StackPanel>
    <user:UserControl1 />
    <user:UserControl2 />
  </StackPanel>
</Window>
```
[UserControl1.xaml]
```xaml
<UserControl x:Class="WpfApplication1.UserControl1"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <TextBlock Text="{Binding Text1}"/>
</UserControl>
```
[UserControl2.xaml]
```xaml
<UserControl x:Class="WpfApplication1.UserControl2"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <TextBlock Text="{Binding Text2}"/>
</UserControl>
```
