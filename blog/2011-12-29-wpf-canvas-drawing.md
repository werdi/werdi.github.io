---
title: Прошу совета по выполнению отрисовки на WPF-Canvas
tags: [c#, xaml, wpf, canvas]
src: https://social.msdn.microsoft.com/Forums/ru-RU/94464ebb-f34a-4ff8-9532-81e7b3a4e1a4/-wpfcanvas?forum=fordesktopru
---
# Прошу совета по выполнению отрисовки на WPF-Canvas
*> Есть ли в WPF какой метод (например у Canvas), с помощью которого можно очищать не всю Canvas, т.е. удалять не все ранее нарисованные элементы на Canvas, а только те которые нужно, т.е. выборочно?*
```xaml
<Window
	xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
	x:Class="WpfApplication5.MainWindow"
	x:Name="Window" Title="MainWindow" Width="640" Height="480">
  <Grid>
    <DockPanel VerticalAlignment="Top" HorizontalAlignment="Left">
      <Button Content="Add" Click="Add_Click" />
      <Button Content="Remove" Click="Remove_Click" />
    </DockPanel>
    <ItemsControl ItemsSource="{Binding}" Margin="0,30,0,0">
      <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
          <Canvas />
        </ItemsPanelTemplate>
      </ItemsControl.ItemsPanel>
    </ItemsControl>
  </Grid>
</Window>
```
```c#
using System.ComponentModel;
using System.Linq;
using System.Windows;
using System.Windows.Media;
using System.Windows.Shapes;

namespace WpfApplication5
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      this.InitializeComponent();
      this.DataContext = new BindingList<Shape>();
    }
    private void Remove_Click(object sender, RoutedEventArgs e)
    {
      var bl = this.DataContext as BindingList<Shape>;
      if (bl.Count > 0) bl.RemoveAt(0);
    }
    private void Add_Click(object sender, RoutedEventArgs e)
    {
      var bl = this.DataContext as BindingList<Shape>;
      var ll = bl.LastOrDefault() as Line;
      var x = ll != null ? ll.X1 + 20 : 10;
      bl.Add(new Line() { X1 = x, Y1 = 10, X2 = x, Y2 = 300, StrokeThickness = 10, Stroke = Brushes.Red });
    }
  }
}
```
