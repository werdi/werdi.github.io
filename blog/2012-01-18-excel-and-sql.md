---
title: Excel + SQL
tags: [c#, winforms, excel, ssce, linq, oledb]
src: https://social.msdn.microsoft.com/Forums/ru-RU/8e6660bd-5916-46eb-ba97-c5e80f04ba4b/how-to-save-data-to-db-using-sqlce-and-ei?forum=sqlce
---
# Excel + SQL
*> Мой план создать лист, создать проект SQL server в visual studio 2010. Языком запросов SQL создать базу, потом на её основе создать образ БД в приложении. [...] Я прошу Вас показать как указать путь на лист и универсальный шаблон.*

т.е. надо программно создать лист в существующем excel-файле?
какую базу данных надо создать: SQL Server (.mdb) или SQL Server Compact (.sdf)?

*> А что Вы посоветуете для внутренней базы для приложения, с дальнейшем возможности размещение её на сервере*

[SQL Server Compact] (http://www.microsoft.com/download/en/details.aspx?id=17876) (ssce) - бесплатная встраиваемая БД для приложений и веб-сайтов.

*> Я изучаю синтаксис запросов SQL и C#*

в ssce есть поддержка linq, т.е. во многих случаях sql знать не обязательно.
ниже пример, который создает базу данных, также привязывает данные в редактору.
 
для работы примера к проекту надо подключить сборки: 
System.Data.SqlServerCe и System.Data.Linq
```c#
using System.Data;
using System.Data.Linq;
using System.Data.Linq.Mapping;
using System.Data.SqlServerCe;
using System.Drawing;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var cs = new SqlCeConnection(@"Data Source='c:\temp\test.sdf'");
      var data = new Data(cs);
      if (data.DatabaseExists() == false)
      {
        data.CreateDatabase();
        data.Items.InsertOnSubmit(new Item { Text = "test" });
        data.SubmitChanges();
      }

      var bs = new BindingSource { DataSource = data.Items };
      bs.AddingNew += (s, e) =>
      {
        var ni = new Item { Text = "new" };
        data.Items.InsertOnSubmit(ni);
        e.NewObject = ni;
      };

      this.Size = new Size(600, 350);
      var dg = new DataGridView
      {
        Parent = this,
        Dock = DockStyle.Fill,
        DataSource = bs,
        AllowUserToAddRows = false,
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill
      };
      dg.Columns["Id"].Visible = false;

      new BindingNavigator(bs) { Parent = this, Dock = DockStyle.Top };

      this.FormClosing += (s, e) => data.SubmitChanges();
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Filter'n'Count", delegate
      {
        // v1
        var res1 = data.Items
          .Where(itm => itm.Id % 2 == 0)
          .Select(itm => itm.Text.Length);

        // v2: другой синтаксис v1
        var res2 = from x in data.Items
               where x.Id % 2 == 0
               select x.Text.Length;

        MessageBox.Show("result: " + res1.Sum() + " " + res2.Sum());
      });
    }
  }

  [Table(Name = "Items")]
  public class Item
  {
    [Column(IsPrimaryKey = true, IsDbGenerated = true)]
    public int Id { get; set; }
    [Column(CanBeNull = false)]
    public string Text { get; set; }
  }

  [Database(Name = "Test")]
  public class Data : DataContext
  {
    public Data(IDbConnection dc) : base(dc) { }
    public Table<Item> Items { get { return this.GetTable<Item>(); } }
  }
}
```
*> Я открываю программу создаю Книгу1 в Лист1 «ключевые данные» происходит экспорт и возможно ещё в Лист2 «все данные, чтобы были». [...] как соединить лист с проект.*

если надо загрузить данные из листа excel, то 
примерно так:
```c#
using System.Collections.Generic;
using System.Data;
using System.Data.OleDb;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var path = @"D:\test.xls";
      var dt = new DataTable();
      dt.Load(LoadData(path, "Sheet1$"));

      new DataGrid { Parent = this, Dock = DockStyle.Fill, DataSource = dt };
    }

    OleDbDataReader LoadData(string path, string sheet)
    {
      var cs = "provider=Microsoft.Jet.OLEDB.4.0; Data Source='" + path + "'; Extended Properties=Excel 8.0;";
      var c = new OleDbConnection(cs);
      c.Open();
      var cmd = c.CreateCommand();
      cmd.CommandText = "select * from  [" + sheet + "]";
      return cmd.ExecuteReader(CommandBehavior.CloseConnection);
    }
  }
}
```
если идентификаторы листов неизвестны, то получить их список  
можно с помощью метода GetOleDbSchemaTable.
```c#   
using System.Data;
using System.Data.OleDb;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    DataTable LoadSchema(string path)
    {
      var cs = "provider=Microsoft.Jet.OLEDB.4.0; Data Source='" + path + "'; Extended Properties=Excel 8.0;";
      var c = new OleDbConnection(cs);
      c.Open();
      try
      {
        return c.GetOleDbSchemaTable(OleDbSchemaGuid.Tables, null);
      }
      finally
      {
        c.Close();
      }
    }
    public Form1()
    {
      var dt = LoadSchema(@"D:\test.xls");
      foreach(DataColumn dc in dt.Columns)
        if(dc.ColumnName != "TABLE_NAME") dc.ColumnMapping = MappingType.Hidden;
      new DataGridView { 
        Parent = this, AllowUserToAddRows=false, Dock = DockStyle.Fill, DataSource = dt };
    }
  }
}
```
