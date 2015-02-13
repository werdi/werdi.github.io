---
title: Binding of two listboxes
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/38d76da6-7c6a-40af-a47e-8aab49b93ebb/binding-of-two-listboxes?forum=wpf
---
# Binding of two listboxes
*> two listboxes are displayed, but the trick is, the user can select different entries in the first listbox, and depending on the selected index, different entries should be displayed in the second listbox.*

if i understood correctly, you need master/details scenario.
below is an example.
```xaml
<Window x:Class="WpfApplication8.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525" FontSize="16">
  <StackPanel>
    <ListBox x:Name="cb" ItemsSource="{Binding Items1}" DisplayMemberPath="Id" IsSynchronizedWithCurrentItem="True" />
    <ListBox ItemsSource="{Binding ElementName=cb,  Path=SelectedItem.Childs}" />
    <ComboBox SelectedIndex="{Binding ElementName=cb, Path=SelectedIndex, Mode=OneWay}" ItemsSource="{Binding Items2}" />
  </StackPanel>
</Window>
```
```c#
using System.Windows;
using System.Windows.Controls;
using System.Windows.Documents;

namespace WpfApplication8
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new
      {
        Items1 = new[] 
        { 
          new { Id=1, Childs=new [] { 11, 12, 13 } },
          new { Id=2, Childs=new [] { 21, 22, 23 } },
        },
        Items2 = new[] { 'a', 'b' }
      };
    }
  }
}
```
*> The listBox2, displayes the right subCategories, but the text for listbox1, gives garbage. Can anyone help me out in getting the Category string displayed in listbox1?*

try the following example. the code is the same as yours, but with some refactoring
and with the correct bindings.
```xaml
<Window x:Class="WpfApplication8.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525" FontSize="16">
  <StackPanel>
    <ListBox x:Name="cb" ItemsSource="{Binding}" DisplayMemberPath="Category" IsSynchronizedWithCurrentItem="True" />
    <ListBox ItemsSource="{Binding ElementName=cb,  Path=SelectedItem.SubCategories}" />
  </StackPanel>
</Window>
```
```c# 
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Documents;

namespace WpfApplication8
{
  public partial class MainWindow : Window
  {
    public class SomeClass
    {
      public string Category { set; get; }
      public List<string> SubCategories { set; get; }
    }

    public class SomeList : ObservableCollection<SomeClass> { }

    public MainWindow()
    {
      InitializeComponent();
      var someObject1 = new SomeClass
      {
        Category = "categ_1",
        SubCategories = new List<string> { "subCateg_1", "subCateg_2", "subCateg_3" }
      };
      var someObject2 = new SomeClass
      {
        Category = "categ_2",
        SubCategories = new List<string> { "subCateg_4", "subCateg_5", "subCateg_6" }
      };

      this.DataContext = new SomeList { someObject1, someObject2 }; ;
    }
}
```