---
title: How to Bind SelectedItem Into the ListView?
tags: [c#, xaml, wpf, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a99a516b-ce3b-4d0b-8a54-a0dcf9eb33f1/how-to-bind-selecteditem-into-the-listview?forum=wpf
---
# How to Bind SelectedItem Into the ListView?
*> my sample is not working still if you have any other workaround for it then please tell me I happy to take forward..*

try using following code. it forks fine.
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525">

  <StackPanel>
    <WrapPanel>
      <Button Content="Prev" Click="Prev_Click" />
      <Button Content="Next" Click="Next_Click" />
    </WrapPanel>
    <ListView ItemsSource="{Binding Items}" SelectedItem="{Binding SelectedItem}"
      DisplayMemberPath="Name">
    </ListView>
  </StackPanel>
</Window>
```
```c#
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Linq;
using System.Windows;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      var data = LoadData();
      this.DataContext = new ViewModel { Items = data, SelectedItem = data[0] };
    }

    IList<object> LoadData()
    {
      return new ObservableCollection<object>(
        Enumerable.Range(0, 10).Select(i => new { Name = "item" + i }));
    }

    private void Next_Click(object sender, RoutedEventArgs e)
    {
      var vm = (this.DataContext as ViewModel);
      var i = vm.Items.IndexOf(vm.SelectedItem);
      if (i + 1 < vm.Items.Count)
        vm.SelectedItem = vm.Items[i + 1];
    }
    private void Prev_Click(object sender, RoutedEventArgs e)
    {
      var vm = (this.DataContext as ViewModel);
      var i = vm.Items.IndexOf(vm.SelectedItem);
      if (i - 1 >= 0)
        vm.SelectedItem = vm.Items[i - 1];
    }
  }

  public class ViewModel : INotifyPropertyChanged
  {
    public IList<object> Items { get; set; }

    public object SelectedItem
    {
      get { return _SelectedItem; }
      set
      {
        _SelectedItem = value;
        OnPropertyChanged("SelectedItem");
      }
    }
    private object _SelectedItem;

    private void OnPropertyChanged(string p)
    {
      this.PropertyChanged(this, new PropertyChangedEventArgs(p));
    }
    public event PropertyChangedEventHandler PropertyChanged = delegate { };
  }
}
```