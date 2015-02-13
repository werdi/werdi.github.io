---
title: Про LINQ to SQL и INNER JOIN
tags: [linq, sql, dbml, asp, c#]
src: http://mindberg.blogspot.ru/2008/01/linq-to-sql-inner-join.html
---
# Про LINQ to SQL и INNER JOIN
Request:
```sql
[Id] [uniqueidentifier] NOT NULL
[Date] [datetime] NOT NULL
```
Form:
```sql
[RequestId] [uniqueidentifier] NOT NULL,
[Key] [nvarchar](10) COLLATE Cyrillic_General_CI_AS NULL,
[Value] [nvarchar](4000) COLLATE Cyrillic_General_CI_AS NOT NULL,
[Id] [uniqueidentifier] NOT NULL,
```
Ограничение:
```sql
CONSTRAINT [Request_Form] FOREIGN KEY([RequestId]) REFERENCES [dbo].[Request]([Id])
```
В переводе на человеческий: есть таблица запросов Request, в которой хранятся идентификаторы и даты, а также есть таблица Form, 
в которой хранятся пары Key и Value, при этом для одного Request может быть несколько строк в Form.

Например, надо из Form выбрать 20-ть Value, которым соответствуют Key = “mail”, и также значение Date из родительского Request.

На SQL этот запрос выглядит следующим образом:
```sql
SELECT TOP (20) [t1].[Date], [t0].[Value] AS [Mail]
FROM [Form] AS [t0]
INNER JOIN [Request] AS [t1] ON [t1].[Id] = [t0].[RequestId]
WHERE [t0].[Key] = @p0
```
Но не надо бежать и изучать SQL с его «страшными» INNER JOIN'ами. Можно использовать LINQ to SQL (часть язык C# 3.0), который SQL-запрос сформирует сам. Вот как он выглядит:
```c#
var mails = from frm in dc.Form
  where frm.Key == "mail"
  select new { Date = frm.Request.Date, Mail = frm.Value };
```
где dc – это экземпляр наследника DataContext'а. 

Пример, использования в ASP.NET:
```asp
<asp:DataGrid runat="server" ID="_MailList" AutoGenerateColumns="true" Width="100%"
  BorderWidth="0" AlternatingItemStyle-BackColor="#e0e0e0">
</asp:DataGrid>
…
```
```c#
_MailList.DataSource = mails.Take(20);
_MailList.DataBind();        // запрос к SQL Server’у
```
Наследник DataContext’а формируется автоматически с помощью MSLinqToSQLGenerator'а на основе следующего .dbml-файла (далее фрагмент):
```xml
<Table Name="" Member="Form">
  <Type Name="Form">
    <Column Name="RequestId" Type="System.Guid" CanBeNull="false" />
    <Column Name="Key" Type="System.String" DbType="nvarchar(10)" CanBeNull="false" />
    <Column Name="Value" Type="System.String" CanBeNull="false" />
    <Column Name="Id" Type="System.Guid" IsPrimaryKey="true" CanBeNull="false" />
    <Association Name="Request_Form" Member="Request" ThisKey="RequestId" Type="Request" IsForeignKey="true" />
  </Type>
</Table>
<Table Name="" Member="Request">
  <Type Name="Request">
    <Column Name="Id" Storage="_Id" Type="System.Guid" IsReadOnly="true" IsPrimaryKey="true" CanBeNull="false" />
    <Column Name="Date" Type="System.DateTime" IsReadOnly="true" CanBeNull="false" />
    <Association Name="Request_Form" Member="Form" OtherKey="RequestId" Type="Form" />
  </Type>
</Table>
```
Знать .dbml-формат необязательно, потому что в Visual Studio 2008 есть графический редактор, с помощью которого достаточно легко создать схему БД и отношения между таблицами.

Кстати, у DataContext есть метод CreateDatabase(), который предназначен для создания файлов БД (.mdb-файл  - SQL Server; для SQL Server CE не проверял, но вроде тоже поддерживает) на основе .dbml-файла.

Производители клавиатур терпят убытки, а жизнь у меня и коллег становится немного легче.
