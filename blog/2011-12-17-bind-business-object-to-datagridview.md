---
title: Bind business object to datagridview
tags: [c#, sql, winforms, database]
src: https://social.msdn.microsoft.com/Forums/ru-RU/cd820ef8-408d-49ef-a1a8-ba3512ca626e/bind-business-object-to-datagridview?forum=winformsdatacontrols 
---
# Bind business object to datagridview
*> I am developing with visual studio 2005. [...] I need i.e Company1 Company2 Company3 . . . CompanyN Person1 1 2 3 . . . 1 ...*

seems you need to rotate rows into columns. for this you can use pivot operator from sql server 2005 or later.
let's suppose there is a database 'DataStore' with a following table:

```sql
create table [test]([person] nvarchar(30), [company] nvarchar(30), [priority] int)

insert [test]([person], [company], [priority]) values 
('P1','C1',1), ('P1','C2',2), ('P1','C3',3), ('P2','C1',3), ('P2','C2',2), ('P2','C3',1);
```  
and if you need a following result in the datagridview:
 
``` 
   	C1 C2	C3
P1	1	 2	3
P2	3  2	1    
```
then try using a following code:

```c#
using System;
using System.Data;
using System.Data.SqlClient;
using System.Drawing;
using System.Text;
using System.Windows.Forms;

namespace WindowsFormsApplication6
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new Size(500, 300);

      var dt = new DataTable();
      var sc = new SqlConnection(
          @"Data Source=.\SQLEXPRESS;Initial Catalog=DataStore;Integrated Security=true;");

      var sb = new StringBuilder();
      var delimiter = ",";
      ExecuteReader(sc,
        "SELECT distinct [company] FROM [test]",
        r => 
        {
          while(r.Read())
            sb.Append("[").Append(r.GetValue(0)).Append("]").Append(delimiter);
        });

      if(sb.Length > 1)
      {
        sb.Length -= delimiter.Length;
        ExecuteReader(sc,
          @"SELECT * FROM [test] PIVOT (min([priority]) FOR [company] IN (" + sb + ")) as pvt",
          r => dt.Load(r));
      }

      new DataGridView { Parent = this, Dock = DockStyle.Fill, DataSource = dt };
    }

    Exception ExecuteReader(SqlConnection sc, string sql, Action<IDataReader> callback)
    {
      var c = new SqlCommand { CommandText = sql, Connection = sc };
      try
      {
        sc.Open();
        using (var r = c.ExecuteReader()) callback(r);
        return null;
      }
      catch (Exception exc) { return exc; }
      finally { sc.Close(); }
    }
  }
}
```