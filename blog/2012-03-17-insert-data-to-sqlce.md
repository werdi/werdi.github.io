---
title: No data is inserted to sqlce on resultset.insert() does it require primary key record...?
tags: [c#, sqlce, sdf, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/35ceff73-4d1b-4db0-9437-ee5dd29a614b/no-data-is-inserted-to-sqlce-on-resultsetinsert-does-it-require-primary-key-record?forum=sqlce
---
# No data is inserted to sqlce on resultset.insert() does it require primary key record...?
*> The insert() is not throwing any errors or exceptions.... but no data is being saved and how to know how many number of records are affected...*

try using the following example.
```c#
using System.Data;
using System.Data.SqlServerCe;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var path = @"c:\temp\test1.sdf";
      Create(path);

      var cc = new SqlCeConnection("Data Source=" + path);
      var cm = cc.CreateCommand();
      cm.CommandType = System.Data.CommandType.TableDirect;
      cm.CommandText = "test";
      cc.Open();
      try
      {
        var rs = cm.ExecuteResultSet(ResultSetOptions.Scrollable | ResultSetOptions.Updatable);
        try
        {
          var ti = rs.GetOrdinal("text");
          var ii = rs.GetOrdinal("id");
          for (int i = 0; i < 5; i++)
          {
            var re = rs.CreateRecord();
            re.SetString(ti, "hello" + i);
            rs.Insert(re, DbInsertOptions.PositionOnInsertedRow);
            var id = rs.GetValue(ii);
            System.Diagnostics.Trace.WriteLine(id);
          }
        }
        finally
        {
          rs.Close();
        }
      }
      finally
      {
        cc.Close();
      }
    
      var dt = new DataTable();
      var ca = new SqlCeDataAdapter("select * from [test]", cc);
      ca.Fill(dt);
      new DataGridView {Dock = DockStyle.Fill, Parent = this, DataSource = dt };
    }
    
    private static void Create(string path)
    {
      if (System.IO.File.Exists(path) == false)
      {
        var ce = new SqlCeEngine("Data Source=" + path);
        ce.CreateDatabase();
        var cc = new SqlCeConnection("Data Source=" + path);
        var cm = cc.CreateCommand();
        cm.CommandText = "create table [test] (id int identity, text nvarchar(100))";
        try
        {
          cc.Open();
          cm.ExecuteNonQuery();
        }
        finally
        {
          cc.Close();
        }
      }
    }
  }
}
```
