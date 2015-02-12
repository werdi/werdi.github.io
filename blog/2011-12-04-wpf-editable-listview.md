---
title: WPF Editable Listview
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b753a7ac-63a4-4ab6-8133-3c3fde80ad32/wpf-editable-listview?forum=wpf
---
# WPF Editable Listview
*> 1. First row in listview should allow user to add new item to listview 2. After entering content in first row and click add button, content should be added to listview. 3. except first row other rows in listview should be readonly.*
```xaml
<Window x:Class="WpfApplication8.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication8"
  Title="MainWindow" Height="350" Width="700" FontSize="16">
  <Window.Resources>
    <local:Model x:Key="model" />
  </Window.Resources>
  <ListView DataContext="{Binding Source={StaticResource model}}" ItemsSource="{Binding Items}" x:Name="lv">
    <ListView.Resources>
      <DataTemplate x:Key="ht">
        <Grid>
          <TextBlock Text="{Binding}" />
          <TextBox Margin="0,20,0,0" x:Name="tb" />
        </Grid>
      </DataTemplate>
      <Style TargetType="GridViewColumnHeader">
        <Setter Property="HorizontalContentAlignment" Value="Stretch" />
        <Style.Triggers>
          <Trigger Property="Role" Value="Padding">
            <Setter Property="Template">
              <Setter.Value>
                <ControlTemplate TargetType="GridViewColumnHeader">
                  <Button Content="Add" VerticalAlignment="Bottom" HorizontalAlignment="Left" Width="100" 
                    Margin="2,0,0,2" 
                    Command="{Binding Add}" CommandParameter="{Binding ElementName=lv}" />
                </ControlTemplate>
              </Setter.Value>
            </Setter>
          </Trigger>
        </Style.Triggers>
      </Style>
    </ListView.Resources>
    <ListView.View>
      <GridView>
        <GridView.Columns>
          <GridViewColumn Header="Id" HeaderTemplate="{StaticResource ht}" Width="100" DisplayMemberBinding="{Binding Id}" />
          <GridViewColumn Header="Value" HeaderTemplate="{StaticResource ht}" Width="150" DisplayMemberBinding="{Binding Value}" />
        </GridView.Columns>
      </GridView>
    </ListView.View>
  </ListView>
</Window>
```
```c#
using System;
using System.ComponentModel;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Input;

namespace WpfApplication8
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }
  }

  public class Model
  {
    public ICommand Add { get; private set; }

    public Model()
    {
      this.Add = new AddCommand { Model = this };
      this.Items = new BindingList<object>
      {
        new { Id=0, Value = "v0" },
        new { Id=1, Value = "v1" },
      };
    }
    public object Items { get; set; }
  }

  class AddCommand : ICommand
  {
    public Model Model { get; internal set; }
    public event EventHandler CanExecuteChanged = delegate { };
    public bool CanExecute(object parameter) { return true; }
    public void Execute(object parameter)
    {
      var lv = parameter as ListView;
      // ... 
      var bl = ((BindingList<object>)this.Model.Items);
      bl.Add(new { Id=bl.Count, Value = "v"+bl.Count });
    }
  }
}
```
below is another approach. it makes a gap between column headers and the items. and puts the fields and Add button into the gap.
```xaml
<Window x:Class="WpfApplication8.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication8"
  Title="MainWindow" Height="350" Width="700" FontSize="16">
  <Grid>
    <ListView ItemsSource="{Binding}" x:Name="lv">
      <ListView.Resources>
        <Style TargetType="GridViewColumnHeader">
          <Setter Property="Margin" Value="0,0,0,35" />
          <Setter Property="Height" Value="30" />
        </Style>
      </ListView.Resources>
      <ListView.View>
        <GridView>
          <GridView.Columns>
            <GridViewColumn Header="Id"  Width="100"  DisplayMemberBinding="{Binding Id}" />
            <GridViewColumn Header="Value" Width="150" DisplayMemberBinding="{Binding Value}" />
          </GridView.Columns>
        </GridView>
      </ListView.View>
    </ListView>
    <WrapPanel Margin="3,32,0,0" Height="30" VerticalAlignment="Top">
      <TextBox Width="{Binding ElementName=lv, Path=View.Columns[0].Width}" Margin="1,0,0,0" x:Name="id" Text="new id" />
      <TextBox Width="{Binding ElementName=lv, Path=View.Columns[1].Width}" Margin="1,0,0,0" x:Name="value" Text="new value" />
      <Button Content="Add" Margin="1,0,0,0" VerticalAlignment="Center" Click="Button_Click" />
    </WrapPanel>
  </Grid>
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

      this.DataContext = new BindingList<object>
      {
        new { Id=0, Value = "v0" },
        new { Id=1, Value = "v1" },
      };
    }
    private void Button_Click(object sender, RoutedEventArgs e)
    {
      var bl = ((BindingList<object>)this.DataContext);
      bl.Add(new { Id = bl.Count + ", " + id.Text, Value = "v" + bl.Count + ", " + value.Text });
    }
  }
}
```