---
title: Windows forms databinding
tags: [c#, winforms, database]
src: https://social.msdn.microsoft.com/Forums/ru-RU/f3c51598-3976-41b2-b49d-0b6e8de005ba/windows-forms-databinding?forum=adodotnetentityframework
---
# Windows forms databinding
*> In this I am unable to add any records/modify any values in datagrid. datagrid not allowing me to add new rec. [...] dataGridView1.DataSource = q.ToList();*

instead of ToList try to use ToBindingList
below is an example
```c#
using System.Data.Entity;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new SchoolData();
      if (ds.Database.CreateIfNotExists())
      {
        for (int i = 0; i < 5; i++)
          ds.Schools.Add(new School { Name = "s"+i });
        ds.SaveChanges();
      }
      ds.Schools.Load();
      var dg = new DataGridView 
      { 
        Parent = this, 
        Dock = DockStyle.Fill, 
        DataSource = ds.Schools.Local.ToBindingList() 
      };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("save", (s, e) => ds.SaveChanges());
      this.Menu.MenuItems.Add("add", (s, e) =>  
        ds.Schools.Add(new School { Name = "new" }) );
    }
  }

  public class School
  {
    public int Id { get; set; }
    public string Name { get; set; }
  }
  public class SchoolData : DbContext
  {
    public DbSet<School> Schools { get; set; }
    protected override void OnModelCreating(DbModelBuilder mb)
    {
      base.OnModelCreating(mb);
      mb.Entity<School>().HasKey(t => t.Id).ToTable("Schools");
    }
  }
}
```