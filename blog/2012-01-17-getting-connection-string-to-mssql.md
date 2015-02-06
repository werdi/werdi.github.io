---
title: Получение строки подключения к MSSQL 2008 R2 Express
tags: [c#, winforms, data, sql server]
src: https://social.msdn.microsoft.com/Forums/ru-RU/1ee484ea-9528-4485-89a6-21819f694d3e/-mssql-2008-r2-express?forum=fordataru 
---
# Получение строки подключения к MSSQL 2008 R2 Express
*> где в MSSQL Server 2008 R2 Express взять строку подключения к определённой базе?*

в Visual Studio - Server Explorer - SQL Server - имя_системы\имя_sql_server.
там же есть список Databases. 
например, для базы данных TestData в SQLEXPRESS строка будет:
```
var str = "Server=.\\SQLEXPRESS;Database=TestData;Integrated Security=SSPI";
```
*> как проверить успешность подлючения? [...] Есть ли у DataContext такой метод (свойство?) ?*

можно вызвать DataContext.DatabaseExists();

если используется DataContext, то строка подключения не нужна.
только имя сервера. ниже пример, который подключается к серверу и создает базу данных "TestData" с таблицей Items.
```c#
using System.Data.Linq;
using System.Data.Linq.Mapping;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dc = new Data(".\\SQLEXPRESS");
      if (dc.DatabaseExists() == false)
        dc.CreateDatabase();
    }
  }


  [Table(Name = "Items")]
  class Item
  {
    [Column(IsPrimaryKey = true, IsDbGenerated = true)]
    public int Id { get; set; }
  }

  [Database(Name = "TestData")]
  class Data : DataContext
  {
    public Data(string cs) : base(cs) { }
    public Table<Item> Items { get; set; }
  }
}
```
*> if (db.DatabaseExists()) [...] В случае успешного подключения к серверу должен показать табличку "Ня"? *

да, но при условии, что база данных существует. иначе в проект можно добавить метод DataContextExt, чтобы проверять соединение так:
```c#
var dc = new Data(".\\SQLEXPRESS");
var res = dc.TestConnection();
...
  
public static class DataContextExt
{
  public static bool TestConnection(this DataContext dc)
  {
    try
    {
      using(var c = dc.Connection)
      {
        c.Open();
        return true;
      }
    }
    catch
    {
      return false;
    }
  }
}
```
