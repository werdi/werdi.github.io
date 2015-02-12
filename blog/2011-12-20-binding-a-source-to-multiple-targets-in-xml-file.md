---
title: Binding a source to multiple targets in an XML file
tags: [c#, xaml, wpf, linq, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/412fe0a0-15bf-4b3f-abfc-5af6c3595952/binding-a-source-to-multiple-targets-in-an-xml-file?forum=wpf
---
# Binding a source to multiple targets in an XML file
*> I am trying to set the TreeViewItem.Header to be a TextBox bound to "Byte[@name=\"Text\"]/@value". If I bind the header directly to "Byte[@name=\"Text\"]/@value" then I get an uneditable string*

try using the following example.
```xaml  
<Window x:Class="WpfApplication4.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525" FontSize="16">
  <StackPanel>
    <StackPanel.Resources>
      <HierarchicalDataTemplate x:Key="xe" ItemsSource="{Binding Elements[item]}">
        <TextBox Text="{Binding Mode=TwoWay, Path=Attribute[title].Value}" />
      </HierarchicalDataTemplate>
      <Style TargetType="TreeViewItem">
        <Setter Property="IsExpanded" Value="True" />
      </Style>
    </StackPanel.Resources>
    <TreeView x:Name="tv" ItemsSource="{Binding Elements[item]}" ItemTemplate="{StaticResource xe}" />
    <Button Click="Button_Click" Content="Test" HorizontalAlignment="Left" />
  </StackPanel>
</Window>
```
```c#
using System.Windows;
using System.Xml.Linq;

namespace WpfApplication4
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = XElement.Parse(@"
      <root>
        <item title='itm1'>
          <item title='itm2' />
          <item title='itm3'>
            <item title='itm4' />
          </item>
        </item>
      </root>");
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      MessageBox.Show(""+this.DataContext);
    }
  }
}
```