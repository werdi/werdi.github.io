---
title: How TO Bind XML data Into Listbox ?
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/664257f0-b07c-44fc-b938-cd086687c1ab/how-to-bind-xml-data-into-listbox-?forum=wpf
---
# How TO Bind XML data Into Listbox ?
*> How TO Bind XML data Into Listbox ?*

if xml loaded into XElement, then try using following example
```c#
using System.Windows;
using System.Xml.Linq;

namespace WpfApplication8
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      this.DataContext = XElement.Parse(@"
        <data>
          <item id='1' data='d1'>i1</item>
          <item id='2' data='d2'>i2</item>
          <item id='3' data='d3'>i3</item>
        </data>");
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication8.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525" FontSize="16">
  <StackPanel>
    <!-- v1 -->
    <ListBox ItemsSource="{Binding Elements[item]}" DisplayMemberPath="Attribute[id].Value"/>
    <!-- v2 -->
    <ListBox ItemsSource="{Binding Elements[item]}">
      <ListBox.ItemTemplate>
        <DataTemplate>
          <TextBlock>
            <TextBlock.Text>
              <MultiBinding StringFormat="id: {0}, value: {1}">
                <Binding Path="Attribute[id].Value" />
                <Binding Path="Value" />
              </MultiBinding>
            </TextBlock.Text>
          </TextBlock>
        </DataTemplate>
      </ListBox.ItemTemplate>
    </ListBox>
  </StackPanel>
</Window>
```