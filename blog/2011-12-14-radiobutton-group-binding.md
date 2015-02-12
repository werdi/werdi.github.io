---
title: Is there any possible way to Bind "RadioButton" group?
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/64765fe5-ff98-445c-879d-798aeecbefff/is-there-any-possible-way-to-bind-radiobutton-group?forum=wpf
---
# Is there any possible way to Bind "RadioButton" group?
*> How to bind the Radio Button group with Boolean property? Is this achiveable ?*

if i understood correctly, you want to bind two radiobuttons with a bool property.
so below is an example.
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication9"
  Title="MainWindow" Height="350" Width="500" FontSize="16">
  <StackPanel>
    <Button Content="Test" Click="Button_Click" HorizontalAlignment="Left" />
    <RadioButton Content="True" IsChecked="{Binding Some}" />
    <RadioButton Content="False" IsChecked="{Binding Some, Converter={local:ValueConverter}}" />
    <TextBlock Text="{Binding Some}" />
  </StackPanel>
</Window>
```
```c# 
using System;
using System.ComponentModel;
using System.Globalization;
using System.Windows;
using System.Windows.Data;
using System.Windows.Markup;

namespace WpfApplication9
{
  public partial class MainWindow : Window, INotifyPropertyChanged
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = this;
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      this.Some = !this.Some;
    }
    public event PropertyChangedEventHandler PropertyChanged = delegate { };
    public bool Some
    {
      get { return _Some; }
      set
      {
        _Some = value;
        this.PropertyChanged(this, new PropertyChangedEventArgs("Some"));
      }
    }
    bool _Some;
	}

	public class ValueConverter : MarkupExtension, IValueConverter
	{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return !(bool)value;
    }
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
      return !(bool)value;
    }
    public override object ProvideValue(IServiceProvider serviceProvider)
    {
      return this;
    }
  }
}
```     
p.s.
WinForms-example is [here](http://social.msdn.microsoft.com/Forums/en-us/winformsdatacontrols/thread/f06bcfbb-5e1a-480e-b91f-bbf3ff025b2a/#718a4636-4f82-4f1b-9236-f74e28cc1e7d)
