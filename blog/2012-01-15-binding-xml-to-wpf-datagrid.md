---
title: Binding XML to WPF Datagrid
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9ce3927d-c1e1-495e-ab0c-ba996ebb1653/binding-xml-to-wpf-datagrid?forum=wpf
---
# Binding XML to WPF Datagrid
*> I have a WPF data grid and an xmlString and am having problems binding it. [...] "{Binding Path=Element[panelCode].InnerText}"*

instead of InnerText use Value. also specify Element[load]
below is an example
```c#
using System.Windows;
using System.Xml.Linq;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = XElement.Load("data.xml");
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="500" Width="900">
  <Grid>
    <DataGrid x:Name="dtgMain" ItemsSource="{Binding Path=Element[load].Elements[panel]}"
        AutoGenerateColumns="False">
      <DataGrid.Columns>
        <DataGridTextColumn Header="PanelCode" Binding="{Binding Path=Element[panelCode].Value}"/>
      </DataGrid.Columns>
    </DataGrid>
  </Grid>
</Window>
