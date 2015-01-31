---
title: WPF label text 
tags: [wpf, c#, xaml, canvas]
src: https://social.msdn.microsoft.com/Forums/ru-RU/55260c1a-17c0-4ffa-97bc-c4cca5239a8c/wpf-label-text?forum=wpf
---
# WPF label text 
*> I have a few labels on my canvas and if their text changes, they stretch to the right.*

could you please provide xaml, which reproduce the issue.
i tried an example below, it works fine. 
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="300" Width="500"
  xmlns:local="clr-namespace:WpfApplication2">
  <Grid>
    <Button Content="Test" Click="Button_Click" HorizontalAlignment="Left" VerticalAlignment="Top" />
    <Canvas Width="300" Height="100" Margin="0,30,0,0" Background="WhiteSmoke">
      <Label Content="Label1" x:Name="l1" Canvas.Left="10" Canvas.Top="10" />
      <Label Content="Label2" x:Name="l2" Canvas.Left="10" Canvas.Top="50" />
    </Canvas>
  </Grid>
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
    }
    private void Button_Click(object sender, RoutedEventArgs e)
    {
      l1.Content += new string('.', 100);
      l2.Content += "" + 1;
    }
  }
}
```
*> It expands to the right of the screen in your example, and I need it to go the other way..*

in this case just use the HorizontalContentAlignment with Width 
```xaml
<Label HorizontalContentAlignment="Right" Width="200" ...
```
*> But it still requires you to attach an event whenever you create a label. [...] lbl.SizeChanged += (s, e) => ...*

almost the same result we can get using just a xaml
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Width="500" SizeToContent="Height">
  <StackPanel>
    <TextBox x:Name="txt" Text="some" Background="Yellow" />
    <Canvas Height="200">
      <Label Content="{Binding ElementName=txt, Path=Text}"
        Canvas.Top="100" Width="180" HorizontalContentAlignment="Right" />
    </Canvas>
  </StackPanel>
</Window>
```
*> I have dynamically generated labels so the xaml won't matter much.*

so, in code just write somethig like this: 
```c#
var lbl = new Label() { 
  Content = "Hello", HorizontalContentAlignment = HorizontalAlignment.Right, Width = 180 };
canvas.Children.Add(lbl);
```
