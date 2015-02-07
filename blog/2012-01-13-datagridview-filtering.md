---
title: DataGridView filtering
tags: [c#, winforms, datagridview, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/aef2d08e-40a9-4fd2-aec3-1ea7bf3d67e4/datagridview-filtering?forum=csharpgeneral
---
# DataGridView filtering
*> How can i filter out those columns? For example i need to filter out all contacts (and their company names), whose name starts with 'K'?*

in your code use "where" with StartsWith("K") method before "select new { ..."
below is an example, which creates database in sql server and filters rows and columns.

```c#
using System.ComponentModel.DataAnnotations;
using System.Data.Entity;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new DataStore();
      if (ds.Database.CreateIfNotExists())
      {
        ds.Infos.Add(new Info { Name = "a1", Contact = "c1" });
        ds.Infos.Add(new Info { Name = "a2", Contact = "c2" });
        ds.Infos.Add(new Info { Name = "b1", Contact = "c3" });
        ds.Infos.Add(new Info { Name = "b2", Contact = "c4" });
        ds.SaveChanges();
      }

      var rows = from row in ds.Infos 
           where row.Name.StartsWith("a")
           select new { Name = row.Name, Id = row.Id };

      new DataGridView
      {
        Parent = this,
        Dock = DockStyle.Fill,
        AllowUserToAddRows = false,
        DataSource = rows.ToList()
      };
    }
  }

  class Info
  {
    [Key]
    public int Id { get; set; }
    public string Name { get; set; }
    public string Contact { get; set; }
  }
  class DataStore : DbContext
  {
      public DbSet<Info> Infos { get; set; }
  }
}
```
*> submenu don't give me a StartsWith() method:/*

try using something like this:
```c#
var res = from stulpelis in sqlResultSet 
      let name = stulpelis["Company Name"] as string
      where name.StartsWith("K")
      select new
      {
        CompanyName = name,
        ContactName = stulpelis["Contact Name"]
      };
```
