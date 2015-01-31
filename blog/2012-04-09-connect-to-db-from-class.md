---
title: Подключение к базе данных из класса
tags: [data binding, c#, sql server, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b6dc04c4-1876-4366-83aa-bbb91f465df1/-?forum=fordesktopru
---
# Подключение к базе данных из класса
*> Пытаюсь подключиться к базе MSSQL, вынося функцию подключения во внешний файл DB.cs [...] возникает проблема [...] как правильно создать экземпляр класса и обратиться к одному из методов.*

примерно так: 
```c#
using System.Data;
using System.Data.SqlClient;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var data = new Data("Server=.\\SQLEXPRESS;Database=TestData;Integrated Security=SSPI");
      var dt = data.Load("select * from [mytable]");    // загрузить данные
      new DataGridView {Parent = this, Dock = DockStyle.Fill, DataSource = dt }; // вывести таблицу на экран
    }
  }

  class Data
  {
    SqlConnection Connection;
    public Data(string cs)
    {
      this.Connection = new SqlConnection(cs);
    }
    public DataTable Load(string sql)
    {
      var ret = new DataTable();
      var cmd = this.Connection.CreateCommand();
      cmd.CommandText = sql;
      this.Connection.Open();
      using (var r = cmd.ExecuteReader(CommandBehavior.CloseConnection))
        ret.Load(r);
      return ret;
    }
  }
}
```
p.s.
для работы с базами данных  существует более простой способ - EF Code First.
пример см. [здесь] (http://social.msdn.microsoft.com/Forums/ru/fordataru/thread/62916518-0bb9-4356-9f90-d64b07f9fff6/#c3aacc05-0e30-466e-b917-b907fdae38a4)
