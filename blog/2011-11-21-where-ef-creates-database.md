---
title: I can't find where EF created the database
tags: [c#, xml, data, asp.net, mvc]
src: https://social.msdn.microsoft.com/Forums/ru-RU/70eb5613-2304-46a1-9506-5fc7a7f00a1e/i-cant-find-where-ef-created-the-database?forum=adodotnetentityframework
---
# I can't find where EF created the database
*> am a new to MVC 3 which uses EF. I have created code and I have verified that the data is being persisited. I just can't find out where the database was created.*

the path to the database could be written in the web.config, connectionsStrings tag

*> But I still cannot find the defaykt SQL database in SQLExpress.*

let's suppose you have ASP.NET MVC solution with a following
[Web.config] 
```xml
<?xml version="1.0"?>
<configuration>
  <connectionStrings>
    <add name="ApplicationServices"
      connectionString="data source=.\SQLEXPRESS;Integrated Security=SSPI;AttachDBFilename=|DataDirectory|aspnetdb.mdf;User Instance=true"
      providerName="System.Data.SqlClient" />
  </connectionStrings>
  ...
```
[Models\Entities.cs]
```c#
using System.Data.Entity;

namespace MvcApplication2.Models
{
  public class Entity
  {
    public int Id { get; set; }
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
```
[Controllers\HomeController.cs] 
```c#
using System.Data;
using System.Data.Entity;
using System.Web.Mvc;

namespace MvcApplication2.Controllers
{
  public class HomeController : Controller
  {
    public ActionResult Index()
    {
      var d = new MvcApplication2.Models.Data();
      ...
```
in order to get the physical path to the database file, you can write the following
```c#
var path = d.GetPath();
```
below is an implementation of the GetPath extension method
```c#   
public static class DbContextHelper
{
  public static string GetPath(this DbContext dc)
  {
    dc.Database.CreateIfNotExists();
    dc.Database.Connection.Open();
    try
    {
      var c = dc.Database.Connection.CreateCommand();
      c.CommandText = "SELECT filename FROM master..sysdatabases WHERE name='" + dc.GetType().FullName + "'";
      c.CommandType = CommandType.Text;
      return (string) c.ExecuteScalar();
    }
    finally
    {
      dc.Database.Connection.Close();
    }
  }
}
```