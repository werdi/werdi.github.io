---
title: Работа с БД sql-server в ASP.NET на EDM или нет?
tags: [c#, winforms, asp]
src: https://social.msdn.microsoft.com/Forums/ru-RU/0bc22bf0-8135-4a19-abd2-21dcd4778e69/-sqlserver-aspnet-edm-?forum=fordataru 
---
# Работа с БД sql-server в ASP.NET на EDM или нет?
*> сделал отдельный проект [...] классы описывающие данные из БД с методами для извлечения и изменения этих данных [...] стоит ли мне пытаться понять EF или же оставить то что сделал?*

оставить.

EF скрывает детали работы с бд -- удобно для создания примеров или простых app, но в реальных проектах приводит к потерям времени из-за подводных камней.

*> нужно ли мне переделать базу данных на более структурированную?*

если задача учебная, то можно привести структуру в соответствие всем типам [нормальных форм] (http://en.wikipedia.org/wiki/Database_normalization).
но на практике схема бд подстравивает под приложение: [нормализация] (http://support.microsoft.com/kb/283878/ru) заканчивается на 2NF или вообще используется NoSQL, потому что "не все так ладно с «квадратной» моделью таблиц/строк/столбцов. Попытка смоделировать иерархические данные может довести до бешенства", см. [Куда идет NoSQL с MongoDB] (http://msdn.microsoft.com/ru-ru/magazine/ee310029.aspx). [Части 2 и 3] (http://msdn.microsoft.com/ru-ru/magazine/ee310029.aspx)

*> сделать реляционную БД (возможно я применю свой проект asp.net как дипломный через год, а там БД явно должна быть реляционная презентована)*

чтобы не ломать голову с нормальными формами можно создать макет в xml и загрузить 
его в DataGrid, чтобы получить таблицы с отношениями.
ниже пример с редактором текста и DataGrid.
```c#
using System;
using System.Data;
using System.Drawing;
using System.IO;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new Size(900, 700);
      this.Font = new System.Drawing.Font("verdana", 10);
      var sc = new SplitContainer { Parent = this, Dock = DockStyle.Fill, SplitterDistance = 400 };
      var tb = new RichTextBox
      {
        Parent = sc.Panel1,
        Dock = DockStyle.Fill,
        AcceptsTab = true,
        Text = "<test>\n\t<item name='itm1'>\n\t\t<sub id='1' />\n\t</item>\n</test>"
      };
      var dg = new DataGrid { Parent = sc.Panel2, Dock = DockStyle.Fill };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Run", delegate
      {
        var ds = new DataSet();
        try
        {
          ds.ReadXml(new StringReader(tb.Text));
          dg.DataSource = ds;
          dg.Focus();
          SendKeys.Send("%({DOWN})");      // Alt+DownArrow 
        }
        catch (Exception exc)
        {
          MessageBox.Show(exc.Message);
        }
      });
    }
  }
}
```
*> В основном у меня UserControl'ы, в которые я и запихиваю данные.*

контролы могут привязываться к источнику данных или к свойствам объектов.
см. [Control.DataBindings] (http://msdn.microsoft.com/ru-ru/library/system.windows.forms.control.databindings.aspx)

*> Возможно получится вместо datagrid'a использовать dataset*

datagrid - это контрол, который используется для отображения/редактирования таблиц и/или dataset'ов.

*> лучше, сканировать папку с файлами и обрабатывая имена рисовать в контроле ссылку на них для скачивания. Или в базе хранить имена и от туда брать?*

в бд. т.к. можно хранить допинфо.

*> Кто мне даст просматривать так просто сервер, где будут хранится файлы.*

asp.net application может сканировать свои папки и файлы. обычно на хостингах это не запрещают.
примерно так:
```c#
var root = new System.IO.DirectoryInfo(MapPath("~/"));
foreach(var fse in root.EnumerateFileSystemInfos())
{
  System.Diagnostics.Trace.WriteLine(fse.Name);
}
```
