---
title: Find a string in XML
tags: [c#, xaml, xml, xpath]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a728aa1c-c6d6-4390-bdf3-ff145823e899/find-a-string-in-xml?forum=csharplanguage
---
# Find a string in XML
*> the XML file i have [...] i want the listbox to display all the servernames that start with server*
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525">
  <ListBox ItemsSource="{Binding Servers}" />
</Window>
```
```c#
using System.Windows;
using System.Xml.Linq;
using System.Linq;
using System.Xml.XPath;
namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = new
      {
        Servers = XElement.Load("..\\..\\XmlFile1.xml")
                    .XPathSelectElements("//Servername")
                    .Where(e => e.Value.ToLowerInvariant().Contains(@"\server"))
                    .Select(e => e.Value)
      };
    }
  }
}
```
