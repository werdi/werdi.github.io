---
title: Popup control looks like regular window
tags: [wpf, xaml, c#, popup]
src: https://social.msdn.microsoft.com/Forums/ru-RU/bc50e540-8cdf-4110-902f-1b418ebc9452/popup-control-look-like-regular-window?forum=wpf
---
# Popup control looks like regular window
*> Is it possible to create a popup in wpf that behaves as windows form? I mean popup window looks like regular window but without minimize, maximize and close button.*
```xaml
<Window x:Class="WpfApplication2.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    Height="300" Width="300" WindowStyle="None" ResizeMode="NoResize">
</Window>
```
if you need show a window border, just remove ResizeMode.
*> When user selects ListViewItem the popup window is popup. I would like to create popup control looks like regular window.*
try using the following code
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication1"
  Title="MainWindow" Height="350" Width="525">
  <ListView ItemsSource="{Binding}">
    <ListView.Resources>
      <Style TargetType="ListViewItem">
          <EventSetter Event="MouseDoubleClick" Handler="ListViewItem_Click" />
      </Style>
    </ListView.Resources>
      <ListView.View>
        <GridView>
          <GridView.Columns>
            <GridViewColumn Header="Value" DisplayMemberBinding="{Binding Value}" />
          </GridView.Columns>
        </GridView>
      </ListView.View>
  </ListView>
</Window>
```
```c#
using System.Windows;
using System.Windows.Input;
namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new[]   // test data
      {
        new { Value = 1 },
        new { Value = 2 },
      };
    }
    private void ListViewItem_Click(object sender, MouseButtonEventArgs e)
    {
      Window wnd = new Window() { ShowInTaskbar = false, Width = 300, Height = 300 };
      wnd.ShowDialog();
    }
  }
}
```
