---
title: ListBox trigger
tags: [c#, xaml, wpf, binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/333ad38d-dc92-4f70-84be-32d60cf215f4/listbox-trigger?forum=wpf
---
# ListBox trigger
*> There is an option when listbox mouseover trigger true to change all the listBoxItem background?*
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="250" FontSize="16">
  <Window.Resources>
    <Style TargetType="ListBoxItem">
      <Style.Triggers>
        <DataTrigger 
          Binding="{Binding IsMouseOver, RelativeSource={RelativeSource AncestorType=ListBox}}" 
            Value="True">
          <Setter Property="Background" Value="WhiteSmoke" />
        </DataTrigger>
      </Style.Triggers>
    </Style>
  </Window.Resources>
  <ListBox ItemsSource="{Binding}" Height="150" BorderThickness="0" />
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

      this.DataContext = new[] { 1, 2, 3, 4, 5 };
    }
  }
}
```
*> I try it but it is not work. It doesn't do any setter Property. :-/ I have Data Template ,is it matter?*

it seems you have missed to set a ListBox ItemSource="{Binding}" also there is a mix of styles.
below is a minimal working example based on your code.

```c#
using System;
using System.IO;
using System.Windows;

namespace WpfApplication2
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new[] 
      {
        new { Subject = "s1", Text="t1", From = new [] {1,2,3}, Date="d1",  Attachments=new [] {1, 2}},
        new { Subject = "s2", Text="t2", From = new [] {4,5}, Date="d2",  Attachments=new [] {1}},
      };
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="250" Width="750" FontSize="16">
  <Window.Resources>
    <Style TargetType="ListBoxItem">
      <Style.Triggers>
        <DataTrigger 
          Binding="{Binding IsMouseOver, RelativeSource={RelativeSource AncestorType=ListBox}}" 
            Value="False">
          <Setter Property="Opacity"  Value="0.5" />
        </DataTrigger>
      </Style.Triggers>
    </Style>
  </Window.Resources>
  <ListBox x:Name="MailList" ItemsSource="{Binding}">
    <ListBox.ItemTemplate >
      <DataTemplate>
        <TextBlock HorizontalAlignment="Left" Margin="0,0,0,0" x:Name="message">
          <TextBlock Text="{Binding Path=Subject, StringFormat='Subject: {0}; '}" />
          <TextBlock Text="{Binding Path=Text, StringFormat='Text: {0}; '}" />
          <TextBlock Text="{Binding Path=From[0], StringFormat='From: {0}; '}" />
          <TextBlock Text="{Binding Path=Date, StringFormat='Date: {0}; '}" />
          <TextBlock Text="{Binding Path=Attachments.Count, StringFormat='Count: {0}; '}" />
        </TextBlock>
      </DataTemplate>
    </ListBox.ItemTemplate>
  </ListBox>
</Window>
```

*> I  have another Qes. [...] If I want to add an option which when Listitem Is MouseOver = true  to ovveride the Opacity = 1; (Only one, all the rest will stay 0.5) How can I do that?*
```xaml
<Window x:Class="WpfApplication5.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525">
  <Window.Resources>
    <Style TargetType="ListBoxItem">
      <Style.Triggers>
        <DataTrigger 
          Binding="{Binding IsMouseOver, RelativeSource={RelativeSource AncestorType=ListBox}}" 
            Value="True">
          <Setter Property="Opacity"  Value="0.5" />
        </DataTrigger>
        <Trigger Property="IsMouseOver" Value="True">
          <Setter Property="Opacity"  Value="1" />
          <!--<Setter Property="Background"  Value="WhiteSmoke" />-->
        </Trigger>
      </Style.Triggers>
    </Style>
  </Window.Resources>
  <ListBox ItemsSource="{Binding}" />
</Window>
```
