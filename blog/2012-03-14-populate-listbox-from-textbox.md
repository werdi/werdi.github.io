---
title: Populate Listbox from Textbox
tags: [c#, xaml, wpf, listbox]
src: https://social.msdn.microsoft.com/Forums/ru-RU/8075c471-c8b3-4684-b986-19b3ab86fcc6/populate-listbox-from-textbox?forum=wpf
---
# Populate Listbox from Textbox
*> I am trying to create a standalone application which has a textbox , a listbox and a button. Upon clicking the button, I want the listbox to show the entered text in the textbox.*
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525" FontSize="16">
  <StackPanel>
    <TextBox x:Name="tb" Background="Yellow" Text="New Text" />
    <Button Content="Add" Click="Button_Click" HorizontalAlignment="Left" VerticalAlignment="Top" />
    <ListBox ItemsSource="{Binding Items}" />
  </StackPanel>
</Window>
```
```c#
using System.Windows;
using System.Windows.Input;
using System.Windows.Documents;
using System.ComponentModel;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new { Items = new BindingList<string>() };
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      dynamic dc = this.DataContext;
      dc.Items.Add(tb.Text);
      tb.Text = "";
    }
  }
}
```
