---
title: C# 2010 and datacontext object
tags: [c#, asp.net, mvc, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/46967632-c9cc-4289-9743-3d844d26337b/c-2010-and-datacontext-object?forum=linqtosql
---
# C# 2010 and datacontext object
*> if they do not supply a date, the data needs to be set to NULL. Thus in the application how do I pass the null value to the database via the datacontext object? Do I pass the value as '' or 'NULL', or some other value?*

you can use Nullable type; take a look at the public DateTime? Date { get; set; } below.
```c#
using System;
using System.Data.Entity;

namespace MvcApplication2.Models
{
  public class Entity
  {
    public int Id { get; set; }
    public DateTime? Date { get; set; }
  }
  public class Data : System.Data.Entity.DbContext
  {
    public DbSet<Entity> Entities { get; set; }
    protected override void OnModelCreating(DbModelBuilder mb)
    {
      base.OnModelCreating(mb);
      mb.Entity<Entity>().HasKey(t => t.Id).ToTable("Entities");
    }
  }
}
 
using System.Data;
using System.Data.Entity;
using System.Web.Mvc;
using MvcApplication2.Models;

namespace MvcApplication2.Controllers
{
  public class HomeController : Controller
  {
    public ActionResult Index()
    {
      var d = new Data();
      d.Entities.Add(new Entity() { Date = null });
      d.SaveChanges();
      ...
```
*> I am not using the entity framework and MVC. Thus how would this code be differrent to pass a null the database using linq to sql?*

take a look at the following example. at the method Update.
```c#
using System;
using System.Data.Entity;
using System.Data.Linq;
using System.Data.Linq.Mapping;
using System.Linq;

namespace ConsoleApplication1
{
  class Program
  {
    [STAThread]
    static void Main(string[] args)
    {
      Create();
      Update();
    }
    static void Create()
    {
      var d = new Data(@".\SQLEXPRESS");
      if (d.DatabaseExists() == false)
        d.CreateDatabase();

      var et = d.GetTable<Entity>();
      et.InsertOnSubmit(new Entity() { Date = DateTime.Now });
      d.SubmitChanges();
    }
    static void Update()
    {
      var d = new Data(@".\SQLEXPRESS");
      var et = d.GetTable<Entity>();
      var r = et.OrderByDescending(e => e.Id).First();
      r.Date = null;     // 
      d.SubmitChanges();
    }
  }
  [Database(Name = "Test.Data")]
  public class Data : DataContext
  {
    public Data(string path) : base(path) { }
    public Table<Entity> Entities; // for db generation
  }
  [Table(Name = "Entity")]
  public class Entity
  {
    [Column(IsPrimaryKey = true, IsDbGenerated = true)]
    public int Id { get; set; }
    [Column(CanBeNull = true)]
    public DateTime? Date { get; set; }
  }
}
``` 
below is almost the same code for those who uses EF Code First
```c#   
using System;
using System.Data.Entity;
using System.Data.Linq.Mapping;
using System.Linq;

namespace ConsoleApplication1
{
  class Program
  {
    [STAThread]
    static void Main(string[] args)
    {
      Create();
      Update();
    }
    static void Create()
    {
      var d = new Data();
      d.Database.CreateIfNotExists();
      d.Entities.Add(new Entity() { Date = DateTime.Now });
      d.SaveChanges();
    }
    static void Update()
    {
      var d = new Data();
      var r = d.Entities.OrderByDescending(e => e.Id).First();
      r.Date = null;     // 
      d.SaveChanges();
    }
  }
  public class Entity
  {
    public int Id { get; set; }
    public DateTime? Date { get; set; }
  }
  public class Data : DbContext
  {
    public DbSet<Entity> Entities { get; set; }
    protected override void OnModelCreating(DbModelBuilder mb)
    {
      base.OnModelCreating(mb);
      mb.Entity<Entity>().HasKey(t => t.Id).ToTable("Entities");
    }
  }
}
```