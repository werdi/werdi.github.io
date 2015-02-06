---
title: How to save data to Db using SQLce and EI
tags: [c#, ssce, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/8e6660bd-5916-46eb-ba97-c5e80f04ba4b/how-to-save-data-to-db-using-sqlce-and-ei?forum=sqlce
---
# How to save data to Db using SQLce and EI
*> need help with saving new and updated data to the Db.*

an examples are [here] (http://social.msdn.microsoft.com/Forums/en-US/linqtosql/thread/46967632-c9cc-4289-9743-3d844d26337b/#fd25fe6b-21e4-4e37-be34-5504c39a8f38) (SaveChanges and SubmitChanges)

*> I was asking about SQL Server Compact Edition 4.0, not SQL Express.. Do you know how to save changes to CE 4.0 when using an Entity Model?*

an approach is the same. just use SqlCeConnection.
```c#
using System.Data.Common;
using System.Data.Entity;
using System.Data.Linq;
using System.Data.SqlServerCe;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form2 : Form
  {
    public Form2()
    {
      var d1 = new Data1(new SqlCeConnection(@"Data Source=C:\Temp\Test.sdf"));
      var d2 = new Data2(new SqlCeConnection(@"Data Source=C:\Temp\Test.sdf"));
    }
  }

  public class Data1 : DataContext
  {
    public Data1(DbConnection dc) : base(dc) { }
    // ...
  }

  public class Data2 : DbContext
  {
    public Data2(DbConnection dc) : base(dc, true) { }
    // ...
  }
}
```
*> So, you are saying that to insert data into a database that is modeled would be something like this? [...] SQL = "insert data into table...."*

please take a close look at the following [examples] (http://social.msdn.microsoft.com/Forums/en-US/linqtosql/thread/46967632-c9cc-4289-9743-3d844d26337b/#fd25fe6b-21e4-4e37-be34-5504c39a8f38).
(there is no sql, just code, which creates database, inserts and updates data)
