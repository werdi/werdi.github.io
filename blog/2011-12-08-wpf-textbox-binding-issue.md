---
title: WPF TextBox Binding Issue
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/95a57ab7-3219-42ce-9fb1-a49b0dcc12ba/wpf-textbox-binding-issue?forum=wpf
---
# WPF TextBox Binding Issue
*> If I bind TextBox's Text property to a data model [...] The problem comes when I have text in the TextBox and try to use backspace to clear it. Once it is cleared, if I change the focus to a different control, the empty string left in the TextBox is not sent to the data model.*

if you need to update data source on text changed event, then set UpdateSourceTrigger=True;
below is an example.
```xaml
<Window x:Class="WpfApplication8.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication8"
  Title="MainWindow" Height="500" Width="500" FontSize="16">
  <StackPanel>
    <TextBox Text="{Binding Value, UpdateSourceTrigger=PropertyChanged}" />
    <TextBox Text="{Binding Value}" />
  </StackPanel>
</Window>
```
```c# 
using System.ComponentModel;
using System.Windows;

namespace WpfApplication8
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new Model() { Value = "test" };
    }
  }
  public class Model : INotifyPropertyChanged
  {
    public event PropertyChangedEventHandler PropertyChanged = delegate { };
    public string Value
    {
      get { return _Value; }
      set
      {
        _Value = value;
        System.Diagnostics.Trace.WriteLine(value);
        this.PropertyChanged(this, new PropertyChangedEventArgs("Value"));
      }
    }
    string _Value;
  }
}
```