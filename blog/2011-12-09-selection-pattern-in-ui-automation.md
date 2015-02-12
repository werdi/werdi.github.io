---
title: Selection Pattern in UI Automation
tags: [c#, xaml, wpf, linq automation]
src: https://social.msdn.microsoft.com/Forums/ru-RU/6452d442-57a6-4a33-ba91-6c4562d13efe/selection-pattern-in-ui-automation?forum=wpf
---
# Selection Pattern in UI Automation
*> Can anybody explain out the selection pattern in UI Automation.*

take a look at the [Control Patterns Overview](http://msdn.microsoft.com/en-us/library/ms752362.aspx) (there is a SelectionPattern description) 
and [Windows Desktop Development for Accessibility and Automation](http://social.msdn.microsoft.com/Forums/en-us/windowsaccessibilityandautomation/threads)

*> I have read all those, i reached certain level in my codings and am strugglng to invoke a cell in the grid.*

take a look at the following example. hope this helps.
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="500" FontSize="16">
  <StackPanel>
    <Button HorizontalAlignment="Left" Content="Test" Click="Button_Click" />
    <DataGrid CanUserAddRows="False" ItemsSource="{Binding}" x:Name="dg" />
  </StackPanel>
</Window>
```
```c#
using System;
using System.Linq;
using System.Windows;
using System.Windows.Automation;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext =
        Enumerable.Range(0, 5)
          .Select(i => new Info { Id = i, Value = "v" + i })
          .ToArray();
    }
    class Info
    {
      public int Id { get; set; }
      public string Value { get; set; }
    }
    private void Button_Click(object sender, RoutedEventArgs e)
    {
      var ae = GetGrid();
      var gp = ae.GetCurrentPattern(GridPattern.Pattern) as GridPattern;

      var cell = gp.GetItem(1, 1);

      var v = cell.GetCurrentPattern(ValuePattern.Pattern) as ValuePattern;
      var a = cell.GetCurrentPattern(InvokePattern.Pattern) as InvokePattern;
      a.Invoke();

      System.Diagnostics.Trace.WriteLine(v.Current.Value, System.Reflection.MethodBase.GetCurrentMethod().Name);
    }
    AutomationElement GetGrid()
    {
      var p = dg.PointToScreen(new Point(1, 1));
      var ae = AutomationElement.FromPoint(p);
      while (ae != null && ae.Current.ClassName != "DataGrid")
        ae = TreeWalker.ControlViewWalker.GetParent(ae);
      return ae;
    }
  }
}
```