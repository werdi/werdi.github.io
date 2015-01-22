---
title: База данных за 5 минут
tags: [db, linq to sql, xml, dbml, c#]
src: http://cognitex.blogspot.ru/2009/05/5.html
---
# База данных за 5 минут
Например, надо создать Windows приложение, которое позволяет редактировать данные в таблице, которая состоит из двух столбцов. Данные должны сохраняться в базе данных в момент закрытия.
Итак, время пошло. 
<ol>
  <li>Открыть Visual Studio 2008</li>
  <li>Ctrl+Shift+N; Project types: Windows, Templates: Windows Forms Application, OK. В результате в Solution Explorer появится проект, например, WindowsFormsApplication46</li>
  <li>Ctrl+Shift+A; Categories: Data, Templates: LINQ to SQL Classes, Add</li>
  <li>В Solution Explorer стать на DataClasses1.dbml и в контекстном меню выбрать Open With... - Xml Editor - OK. Возможно появится диалог; в нем нажать Yes</li>
  <li>В открывшийся файл вставить следующий xml:</li>
</ol>
```xml
<?xml version="1.0" encoding="utf-8"?>
<Database Class="AppDataContext" xmlns="http://schemas.microsoft.com/linqtosql/dbml/2007">
  	<Table Name="" Member="AppData">
    	<Type Name="AppData">
      	<Column Name="Id" Type="System.Int32" IsPrimaryKey="true" IsDbGenerated="true" CanBeNull="false" />
      	<Column Name="Value" Type="System.String" CanBeNull="false" />
    	</Type>
  	</Table>
</Database>
```
<ol start="6">
  <li>Сохранить DataClasses1.dbml (в результате у файла появится "подфайл" DataClasses1.designer.cs)</li>
  <li>Открыть Form1.cs и заменить его содержимое следующим:</li>
</ol>
```c#
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication46
{
    	public partial class Form1 : Form
    	{
        	public Form1()
        	{
            	// подключиться к БД
            	var dc = new AppDataContext(@"Data Source=..\..\Database.sdf");

            	// если БД не существует ...
            	if (dc.DatabaseExists() == false)
            	{
                	// создать базу и схему
                	dc.CreateDatabase();

                	// создать тестовые данные
                	for (int i = 0; i < 10; i++)
                	{
                    	dc.AppData.InsertOnSubmit(
                        	new AppData() { Id = i, Value = "v" + i.ToString() });
                	}

                	// сохранить данные в БД
                	dc.SubmitChanges();
            	}

            	DataGridView grid = new DataGridView();
            	grid.Dock = DockStyle.Fill;
            	grid.DataSource = from c in dc.AppData select c;
            	grid.Parent = this;
            	this.Shown += delegate
            	{
                	grid.Columns[0].ReadOnly = true;
            	};

            	// в момент закрытия окна
            	this.Closing += delegate
            	{
                	// сохранить изменения в БД
                	dc.SubmitChanges();
            	};
        	}
    	}
}
```
<ol start="7">
  <li>Нажать F5.</li>
</ol>
