---
title: Get Primary Key Properties of a Entity
tags: [c#, database]
src: https://social.msdn.microsoft.com/Forums/ru-RU/6a4afbea-512b-4814-a2ee-1b2e5a4c1430/get-primary-key-properties-of-a-entity?forum=adodotnetentityframework
---
# Get Primary Key Properties of a Entity
*> is it possible to get the properties that are part of the primary key of the entity?*

you can read information from a database using the following method
```c#
foreach(var c in IndexColumns(ds, "MyTable"))
    System.Diagnostics.Trace.WriteLine(c);
...

public IEnumerable<string> IndexColumns(System.Data.Entity.DbContext dc, string tableName)
{
  if (dc.Database.Exists() == false) throw new ArgumentException();
  dc.Database.Connection.Open();
  var dt = dc.Database.Connection.GetSchema("IndexColumns");
  dc.Database.Connection.Close();
  foreach (DataRow row in dt.Rows)
    if (string.Equals((string)row["table_name"], tableName, StringComparison.OrdinalIgnoreCase))
      yield return (string)row["column_name"];
}
```
*> i was thinking in something "lighter" because the key mapping is already inside the edmx right?*

edmx is an instruction for code generator. so at runtime there is no edmx.
but at runtime you can find property with EdmScalarPropertyAttribute
```c#
var key = FindKey<WindowsFormsApplication4.Entity1>();
System.Diagnostics.Trace.WriteLine(key.Name);
...

PropertyInfo FindKey<T>() where T : EntityObject
{
  var et = typeof(T);
  var at = typeof(EdmScalarPropertyAttribute);
  var props = from p in et.GetProperties()
    let attr = Attribute.GetCustomAttribute(p, at) as EdmScalarPropertyAttribute
    select new { Property=p, IsKey = attr != null ? attr.EntityKeyProperty : false };
  return props.Where(p => p.IsKey == true).Select(p => p.Property).FirstOrDefault();
}
```
*> Ok, but this method is used for ObjectContext right with the EntityObject Generator? Thats because im using DbContext Code Generator so i think i cant do that for my generated classes, right?*

let me clarify. you have some following code and you want to get result "Id"?
```c#
class Item
{
  public int Id { get; set; }
  public string Value { get; set; }
}
class DataStore : DbContext
{
  public DbSet items { get; set; }
  protected override void OnModelCreating(DbModelBuilder m)
  {
    base.OnModelCreating(m);
    m.Entity().HasKey(e => e.Id).ToTable("Items");
  }
} 
```
*> My generated code for OnModelCreating is this: ...*

is there any attribute over Id property? maybe [Key] or [KeyAttribute]?
 if so, try using the following code:
 ```c#
 using System.Linq;
...
var ka = GetAttribute<MyClass, KeyAttribute>("Id");
...
static public A GetAttribute<T, A>(string name)
{
  return typeof(T)
    .GetProperty(name)
    .GetCustomAttributes(typeof(A), true)
    .OfType<A>()
    .FirstOrDefault();
}
```
*> AS you can see, there is no attribute like [Key] etc, thats why im curios to know how the Dbset.Find(object[] keys) works, because it know if the keys match with the model...*

try using the following code:
```c#
using System.Data.Metadata.Edm;
using System.Linq;
using System.Reflection;
...

var mw = new MetadataWorkspace(
  new [] { "res://*/" },
  new [] { Assembly.GetExecutingAssembly() });
var tables = mw.GetItems(DataSpace.CSpace);
foreach(var e in tables.OfType<EntityType>())
  System.Diagnostics.Trace.WriteLine(e.Name + ": " + string.Join("\n", e.KeyMembers));
```