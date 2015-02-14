---
title: Creating extension for file in WPF
tags: [c#, xaml, wpf, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/788629f6-e225-44e6-9fa0-44f8666e773b/creating-extension-for-file-in-wpf?forum=wpf
---
# Creating extension for file in WPF
*> I want to save my objects, which can be circle / rectangle or any...... in that extension. How to do that?*
```xaml
<Window x:Class="WpfApplication3.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <StackPanel>
    <DockPanel>
      <Button Content="SaveAndClear" Click="Save_Click" />
      <Button Content="Load" Click="Load_Click" HorizontalAlignment="Left" />
    </DockPanel>
    <ContentControl x:Name="editor">
      <Canvas>
        <Rectangle Canvas.Left="10" Canvas.Top="10" Width="50" Height="50" Fill="Red" />
        <Ellipse Canvas.Left="100" Canvas.Top="100" Width="50" Height="50" Fill="Green" />
      </Canvas>
    </ContentControl>
  </StackPanel>
</Window>
```
```c# 
using System.IO;
using System.Windows;
using System.Windows.Markup;

namespace WpfApplication3
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }

    private void Save_Click(object sender, RoutedEventArgs e)
    {
      using (var file = File.Open(@"c:\temp\file.abc", FileMode.Create))
      {
        XamlWriter.Save(editor.Content, file);
        editor.Content = null;
      }
    }
    private void Load_Click(object sender, RoutedEventArgs e)
    {
      using (var file = File.OpenRead(@"c:\temp\file.abc"))
        editor.Content = XamlReader.Load(file);
    }
  }
}
```
*> I thought for the typical SAVE and SAVE AS Dialog as we see in windows but it should allow the user to save the file in format i.e. .abc How can we do that?*
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  Width="400" Height="400"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <StackPanel>
    <DockPanel>
      <Button Content="SaveAndClear" Click="Save_Click" />
      <Button Content="Load" Click="Load_Click" HorizontalAlignment="Left" />
      <Button Content="Clear" Click="Clear_Click" HorizontalAlignment="Left" />
    </DockPanel>
    <ContentControl x:Name="editor">
      <Canvas>
        <Rectangle Canvas.Left="10" Canvas.Top="10" Width="50" Height="50" Fill="Red" />
        <Ellipse Canvas.Left="100" Canvas.Top="100" Width="50" Height="50" Fill="Green" />
      </Canvas>
    </ContentControl>
  </StackPanel>
</Window>
```
```c#
using System.Windows;
using System.Windows.Forms;
using System.Windows.Markup;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }
    private void Save_Click(object sender, RoutedEventArgs e)
    {
      var d = new SaveFileDialog();
      d.Filter = "abc files (*.abc)|*.abc";
      if (d.ShowDialog() == System.Windows.Forms.DialogResult.OK)
      {
        using (var file = d.OpenFile())
          XamlWriter.Save(editor.Content, file);
        editor.Content = null;
      }
    }
    private void Load_Click(object sender, RoutedEventArgs e)
    {
      var d = new OpenFileDialog();
      d.Filter = "abc files (*.abc)|*.abc";
      if (d.ShowDialog() == System.Windows.Forms.DialogResult.OK)
      {
        using(var file = d.OpenFile())
          editor.Content = XamlReader.Load(file);
      }
    }
    private void Clear_Click(object sender, RoutedEventArgs e)
    {
      editor.Content = null;
    }
  }
}
```