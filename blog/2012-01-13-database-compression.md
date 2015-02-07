---
title: SQL Server Compact 4.0 сжатие базы данных
tags: [c#, winforms, ssce, sql server]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a5b16c2b-3684-4a2e-b97a-20f27b57b35b/sql-server-compact-40-?forum=fordataru
---
# SQL Server Compact 4.0 сжатие базы данных
*> Использую Visual Studio 2010 SP1 и SQL Server Compact 4.0 [...] выполнить сжатие базы.*
```c#
var ssce = new SqlCeEngine(connStr);
ssce.Shrink();   // или Compact(null);
```
см. [SqlCeEngine] (http://msdn.microsoft.com/en-us/library/system.data.sqlserverce.sqlceengine(v=VS.100).aspx) Class

*> База сжимается, идентификаторы начинают свой отчет от последнего значения которое было в этом поле.*

после сжатия надо изменить identity у столбца с помощью запроса
примерно так: "alter table [test] alter column id identity (0,1)"
```c#
using System;
using System.Data;
using System.Data.SqlServerCe;
using System.IO;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var path = Path.GetFullPath("test.sdf");
      File.Delete(path);

      var sc = new SqlCeConnection("Data Source=" + path);
      var se = new SqlCeEngine(sc.ConnectionString);
      se.CreateDatabase();
      sc.Open();
      try
      {
        var cmd = sc.CreateCommand();
        cmd.CommandText = "create table [test] (id int identity(0,1) primary key, value nvarchar(20))";
        cmd.ExecuteNonQuery();

        SaveData(sc);

        this.Menu = new MainMenu();
        this.Menu.MenuItems.Add("Test", Test);

        LoadData(sc);
      }
      finally
      {
        sc.Close();
      }
    }
  
    int SaveDataCount;
  
    private void SaveData(SqlCeConnection sc)
    {
      ++SaveDataCount;
      var cmd = sc.CreateCommand();
      for (int i = 0; i < 10; i++)
      {
        cmd.CommandText = "insert [test] (value) values ('hello" + i + "-" + SaveDataCount + "')";
        cmd.ExecuteNonQuery();
      }
    }
  
    void LoadData(SqlCeConnection sc)
    {
      this.Controls.Clear();
  
      var cmd = sc.CreateCommand();
      cmd.CommandText = "select * from [test]";
  
      var dt = new DataTable();
      dt.Load(cmd.ExecuteReader());
      new DataGridView { Parent = this, Dock = DockStyle.Fill, AllowUserToAddRows = false, DataSource = dt };
    }
  
    void Test(object sender, EventArgs e)
    {
      var path = Path.GetFullPath("test.sdf");
      var sc = new SqlCeConnection("Data Source=" + path);
      sc.Open();
      try
      {
        var cmd = sc.CreateCommand();
        cmd.CommandText = "delete [test]";
        cmd.ExecuteNonQuery();

        var se = new SqlCeEngine(sc.ConnectionString);
        se.Shrink();

        cmd = sc.CreateCommand();
        cmd.CommandText = "alter table [test] alter column [id] identity (0,1)";
        cmd.ExecuteNonQuery();

        SaveData(sc);

        LoadData(sc);
      }
      finally
      {
        sc.Close();
      }
    }
  }
}
```
