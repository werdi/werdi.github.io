---
title: Соединение с БД в EF 4.2, EF4.3
tags: [c#, xml, ssce, sql server compact, ef]
src: https://social.msdn.microsoft.com/Forums/ru-RU/89cc5841-6803-4da0-ae36-7754aa5016cc/-ef-42-ef43?forum=fordataru 
---
# Соединение с БД в EF 4.2, EF4.3
*> public partial class Entities : DbContext { public Entities(string provider, string filePath): base(provider) ...*

в наследнике DbContext можно не указывать конструктор.
см. [здесь] (http://social.msdn.microsoft.com/Forums/en-US/linqtosql/thread/46967632-c9cc-4289-9743-3d844d26337b/#fd25fe6b-21e4-4e37-be34-5504c39a8f38) (второй пример для DbContext)

*> Как тогда передать Provider?*

провайдер указывать не обязательно. можно передать DbConnection. например для работы с sql server compact
пишем:
```c#
var ssce = new TestStore(new SqlCeConnection(@"Data Source=C:\Temp\Test.sdf"));
...
class TestStore : DbContext
{
  public TestStore(DbConnection c) : base(c, true) { }
  public DbSet<Test> Tests { get; set; }
}
```
*> Непонятно правда в чём разница и почему он так себя ведёт если вызывать конструктор родителя с передачей параметра из app.config.*
строку соединения можно указать в app.config 
(указываем полностью, без каких либо {0})
```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
	<connectionStrings>
	<add name="cs" connectionString="Data Source=.\SQLEXPRESS;attachdbfilename=Test.mdb;Integrated Security=True;Connect Timeout=30;User Instance=False;MultipleActiveResultSets=True"			 providerName="System.Data.SqlClient" />
	       ...
```
для использования "cs" в коде пишем:
```c#
var d = new Data("cs");
... 
public class Data : DbContext
{
  public Data(string con)
    : base(con)
  {
  }
  ...
```
*> Базу необходимо выбирать в рантайме*

в такой ситуации .config не подходит.
надо создавать необходимый DbConnection и передавать его в DbContext
