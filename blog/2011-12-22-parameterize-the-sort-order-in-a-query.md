---
title: How to parameterize the sort order in a query?
tags: [c#, linq, sql server]
src: https://social.msdn.microsoft.com/Forums/ru-RU/19f95199-3d95-4669-8798-ce269a0ed4d3/how-to-parameterize-the-sort-order-in-a-query?forum=linqtosql
---
# How to parameterize the sort order in a query?
*> How do I introduce this sorting column parameter in the LINQ to SQL query without writing a different code segment to handle each column sort order?*

see "[Linq OrderBy - how to dynamically pass expressions] (http://social.msdn.microsoft.com/forums/en-US/linqprojectgeneral/thread/ddd18a4a-c67a-4d0c-a07d-3e2180454a52/)"

or you can use Get Method below to create Expression.
```c#
var dc = new DataStore(new SqlCeConnection("Data Source=Test.sdf"));
var res = dc.Tests.OrderBy(Get<Test,string>("Name"));
...
System.Linq.Expressions.Expression<Func<T,K>> Get<T,K>(string pp)
{
  var p = System.Linq.Expressions.Expression.Parameter(typeof(T), "p");
  return System.Linq.Expressions.Expression.Lambda<Func<T, K>>(
    System.Linq.Expressions.Expression.PropertyOrField(p, pp), 
    p);
}
...
[Table(Name = "Tests")]
public class Test
{
  [Column(IsPrimaryKey = true, IsDbGenerated = true)]
  public int Id { get; set; }
  [Column]
  public string Name { get; set; }
}
public class DataStore : DataContext
{
  public Table<Test> Tests;
  public DataStore(IDbConnection dc) : base(dc) { }
}
```
