---
title: Writing data into a database
tags: [c#, ssce, sql server compact, linq, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/c93b3cac-91c6-439e-b0b0-5da2cec6cc3a/writing-data-into-a-database?forum=csharplanguage
---
# Writing data into a database 
*> I need to write a program in c# to store the total times [...] How do I create such a database and how do I go about with integrating it with the program in c#?*

install [sql server compact] (http://nuget.org/packages/SqlServerCompact); create windows forms application; add reference: C:\Program Files\Microsoft SQL Server Compact Edition\v4.0\Desktop\System.Data.SqlServerCe.dll
and try using the following example:
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
      var cs = new SqlCeConnection(@"Data Source='d:\temp\test.data.sdf'");
      var data = new Data(cs);
      if (data.DatabaseExists() == false)
      {
        data.CreateDatabase();
        data.Items.InsertOnSubmit(new Item { Team = "team1", Time = 21.78 });
        data.Items.InsertOnSubmit(new Item { Team = "team2", Time = 20.99 });
        data.SubmitChanges();     // save data
      }

      this.Size = new Size(600, 350);
      new DataGridView { Parent = this, Dock = DockStyle.Fill, DataSource = data.Items.ToList() };
    }
  }


  [Table(Name = "Items")]
  public class Item
  {
    [Column(IsPrimaryKey = true, IsDbGenerated = true)]
    public int Id { get; set; }
    [Column(CanBeNull = false)]
    public string Team { get; set; }
    [Column(CanBeNull = false)]
    public double Time { get; set; }
  }

  [Database(Name = "Test")]
  public class Data : DataContext
  {
    public Data(IDbConnection dc) : base(dc) { }
    public Table<Item> Items { get { return this.GetTable<Item>(); } }
  }
}
```
*> However, I'm unable to install sql server compact because I'm unable to find 'Library Package Manager' under my Visual studio 2010*

direct link to the Microsoft SQL Server Compact 4.0 download page:
http://www.microsoft.com/download/en/details.aspx?id=17876

*> some errors are: [...] does not contain a definition for 'DatabaseExists' [...] The type or namespace name 'Table' could not be found (are you missing a using directive or an assembly reference?)*

you should add reference to the System.Data.Linq.dll assembly.
for this in the Visual Studio main menu select: Project - Add Reference... - .NET: System.Data.Linq
