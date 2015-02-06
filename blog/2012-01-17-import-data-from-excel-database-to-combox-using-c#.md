---
title: Import data from Excel database to combox using C#
tags: [c#, excel, data, winforms, oledb]
src: https://social.msdn.microsoft.com/Forums/ru-RU/7a18613f-1f5a-4f66-963f-9732e0ac090c/import-data-from-excel-database-to-combox-using-c?forum=csharplanguage
---
# Import data from Excel database to combox using C#
*> how to import data from Excel databse*

the following method helps to extract a single column from sheet of Excel
```c#
IEnumerable<string> LoadData(string path, string sheet, string field)
{
  var cs = "provider=Microsoft.Jet.OLEDB.4.0; Data Source='" + path + "'; Extended Properties=Excel 8.0;";
  using (var c = new OleDbConnection(cs))
  {
    c.Open();
    var cmd = c.CreateCommand();
    cmd.CommandText = "select " + field + " from  [" + sheet + "]";
    var res = new List<string>();
    using (var r = cmd.ExecuteReader())
    {
      while (r.Read())
        res.Add(System.Convert.ToString(r.GetValue(0)));
    }
    return res;
  }
}
```
another approach: read all data to the DataTable and bind it to the ComboBox
below is an example
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

      // for test
      new DataGrid { Parent = this, Dock = DockStyle.Fill, DataSource = dt };

      // create and bind combobox; "F3" is a name of the column in the test.xls 
      new ComboBox { Parent = this, Dock = DockStyle.Top, DisplayMember = "F3", DataSource = dt };
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
