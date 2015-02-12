---
title: Change WPF DataGrid SelectedItem based on Textbox input?
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/f25a1b60-c06c-4f5c-99eb-766b3bcb7df5/change-wpf-datagrid-selecteditem-based-on-textbox-input?forum=wpf
---
# Change WPF DataGrid SelectedItem based on Textbox input?
*> if i have more data's in DataGrid its finds very slowSo i have a plan to use DataGrid as Objects.*

with DataView you'll get a fast search.
 below is an example of searching among 10000 rows.
```xaml
<Window x:Class="WpfApplication7.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication7"
  Title="MainWindow" Height="350" Width="525" FontSize="16">
  <Grid>
    <TextBox x:Name="tb" />
    <DataGrid ItemsSource="{Binding}" Margin="0,30,0,0"  
      SelectedIndex="{Binding ElementName=tb, Path=Text, Converter={local:Converter}}" />
  </Grid>
</Window>
```
```c#
using System.Data;
using System.Windows;
using System.Windows.Data;
using System.Windows.Markup;
using System.Xaml;

namespace WpfApplication7
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = View = LoadData();
    }
    private DataView LoadData()
    {
      var dt = new DataTable();
      dt.Columns.Add("Id", typeof(int));
      var n = dt.Columns.Add("Name", typeof(string));
      for (int i = 0; i < 10000; i++) 
        dt.Rows.Add(i, "n" + i);
      dt.DefaultView.Sort = "Name";
      return dt.DefaultView;
    }
    public DataView View;
	}

	public class Converter : MarkupExtension, IValueConverter
	{
    MainWindow Wnd;
    public object Convert(object value, System.Type targetType, object parameter, System.Globalization.CultureInfo culture)
    {
      if(Wnd.View == null) return -1;
      Wnd.View.RowFilter = "Name LIKE '*" + value + "*'";
      return 0;
    }
    public object ConvertBack(object value, System.Type targetType, object parameter, System.Globalization.CultureInfo culture)
    {
      throw new System.NotImplementedException();
    }
    public override object ProvideValue(System.IServiceProvider serviceProvider)
    {
      Wnd = (serviceProvider.GetService(typeof(IRootObjectProvider)) as IRootObjectProvider).RootObject as MainWindow;
      return this;
    }
  }
}
```