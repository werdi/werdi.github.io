---
title: Привязка DataGrid к данным из MSSQL Express
tags: [c#, xaml, wpf, winforms, sql server]
src: https://social.msdn.microsoft.com/Forums/ru-RU/1ceb56d8-62ae-4f09-b2a2-aa79fd96b3ae/-datagrid-mssql-express?forum=fordataru
---
# Привязка DataGrid к данным из MSSQL Express
*> нужно отобразить содержание таблицы в БД [...] есть таблица test, в ней две колонки: number и test_data. [...] по наступлении события Connect_Click() поместить данные их базы данных в DataGrid*

для работы DataContext надо определить соответствующий класс, который будет заполняться данными.
но можно загрузить данные в таблицу и привязать к DataGrid. 
```c#
using System.Data;
using System.Data.SqlClient;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var c = new SqlConnection("Server=.\\SQLEXPRESS;Database=TestData;Integrated Security=SSPI");
      var cmd = c.CreateCommand();
      cmd.CommandText = "select * from [items]";
      c.Open();
      using (var r = cmd.ExecuteReader(CommandBehavior.CloseConnection))
      {
        var dt = new DataTable();
        dt.Load(r);
        new DataGrid { Parent = this, Dock = DockStyle.Fill, DataSource = dt };
      }
    }
  }
}
```
*> Table<dataModify> test = db.GetTable<dataModify>(); //что сюда написать чтобы вывести столбцы num и //value в DataGrid с именем Data?*
```
this.Data.DataContext = test.ToList();
```
в xaml должен быть тег, например, такой:
```xaml
<DataGrid ItemsSource="{Binding}" Name="Data" ColumnWidth="*" />
```
*> как добиться аналогичного эффекта, используя окно свойств элемента?*

в xaml в ресурсах должен быть ObjectDataProvider и т.д. чтобы wpf-редактор знал как получить данные.
ниже пример для подключения к базе данных "TESTDB".
```xaml
<Window x:Class="WpfApplication3.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="300" Width="300"
  xmlns:app ="clr-namespace:WpfApplication3"
  xmlns:sys="clr-namespace:System;assembly=mscorlib"
  xmlns:linq="clr-namespace:System.Data.Linq;assembly=System.Data.Linq">
  <Window.Resources>
    <ObjectDataProvider x:Key="_dc" ObjectType="{x:Type linq:DataContext}">
      <ObjectDataProvider.ConstructorParameters>
        <sys:String>Server=.\SQLEXPRESS;Database=TESTDB;Integrated Security=SSPI</sys:String>
      </ObjectDataProvider.ConstructorParameters>
    </ObjectDataProvider>
    <ObjectDataProvider x:Key="_dt" ObjectInstance="{StaticResource _dc}" MethodName="GetTable">
      <ObjectDataProvider.MethodParameters>
        <x:Type Type="app:dataModify"/>
      </ObjectDataProvider.MethodParameters>
    </ObjectDataProvider>
  </Window.Resources>
  <Grid>
    <DataGrid ItemsSource="{Binding}" Name="Data" ColumnWidth="*"  
        DataContext="{Binding Source={StaticResource _dt}}" />
  </Grid>
</Window>
```
```c#
using System.Data.Linq.Mapping;
using System.Windows;

namespace WpfApplication3
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }
  }

  [Table(Name = "test")]
  public class dataModify
  {
    [Column(IsPrimaryKey = true, IsDbGenerated = true)]
    public int num { get; set; }
    [Column]
    public string value { get; set; }
  }
}
```
