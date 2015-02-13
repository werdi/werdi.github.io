---
title: Организация работы с таблицей ввода
tags: [c#, winforms, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/95600a43-400f-4991-b7cd-75fbac1faa9b/-?forum=fordataru
---
# Организация работы с таблицей ввода
*> Таблица для ввода данных реализована с помощью DataGridView. Часть данных будет вводиться используя ComboBox.*

ComboBox должен выводиться в ячейке DataGridView?

*> Как организовать наполнение этих комбобоксов ?*

см. [ComboBox.DataSource](http://msdn.microsoft.com/en-us/library/x8160f6f.aspx)

*> 1. Как организовать наполнение этих комбобоксов ? 2. Как сохранить введенные в DataGridView данные в таблицу Table (учитывая комбобоксы) ...*

примерно так
```c#
using System.Windows.Forms;
using System.Data;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable("Values");
      dt.Columns.Add("Id", typeof(int)).AutoIncrement = true;
      dt.Columns.Add("Value", typeof(string));

      var ds = new DataSet();
      ds.Tables.Add(dt);

      this.Size = new System.Drawing.Size(600, 400);
      var dg = new DataGridView
      {
        AutoGenerateColumns = false,
        Parent = this,
        Dock = DockStyle.Fill,
        DataSource = dt,
      };
      dg.Columns.Add(new DataGridViewTextBoxColumn 
      {
          DataPropertyName = "Id"
      });
      dg.Columns.Add(new DataGridViewComboBoxColumn 
      { 
        FlatStyle = FlatStyle.Popup,
        DataSource = new [] { "v1", "v2", "v3"},
        DataPropertyName = "Value"
      });

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Save", (s, e) => 
      {
        // отправить на сервер ...
      });
    }
  }
}
```
способы отправки на сервер: 
1) см. DataAdapter - для SQL Server
2) для web-service примерно так: 
```c#
ds.AcceptChanges(); 
var xml = ds.GetXml(); 
var w = new WebClient(); 
Uri url = ...
w.UploadStringAsync(url, xml);
```
