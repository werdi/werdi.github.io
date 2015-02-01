---
title: Конвертация date из mysql в date xml
tags: [c#, mysql, xml, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/311cc8fb-8523-4887-a049-8ffdcd0b7483/-date-mysql-date-xml?forum=fordataru
---
# Конвертация date из mysql в date xml
*> импортирую эту таблицу из mysql в xml, но там эта дата отображается как 2007-02-04T00:00:00+03:00. В чем проблема?*

это такой формат: дата+время+часовой пояс.
если изменить код импортирующей программы невозможно, то можно изменить xml.
как вариант, см. [XStreamingElement] (http://msdn.microsoft.com/ru-ru/library/bb299195.aspx)

*> А можно ли в mysql отключить отображение часового пояса?*

в запросе вместо "select date_service ..." укажите "select [CONVERT] (http://msdn.microsoft.com/en-us/library/ms187928.aspx)(varchar(10), date_service,101) ..." - это работает в sql server; возможно что и в mysql также.

*> в mysql есть такая возможность, но пишет что синтаксическая ошибка*

если на стороне бд не получается, то можно изменить значение в момент записи в файл.
для этого надо создать наследника XmlTextWriter. примерно так:

```c#
using System;
using System.Data;
using System.IO;
using System.Text;
using System.Windows.Forms;
using System.Xml;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      // создать DataSet
      var dt = new DataTable("dates");
      dt.Columns.Add("date", typeof(DateTime));
      dt.Columns.Add("value", typeof(string));
      var ds = new DataSet("org");
      ds.Tables.Add(dt);
      dt.Rows.Add(DateTime.Now, "v");

      // сохранить DataSet в xml
      using(var x = new XmlWriter("result.xml"))
        ds.WriteXml(x, XmlWriteMode.IgnoreSchema);

      new RichTextBox { Parent = this, Dock = DockStyle.Fill, Text = File.ReadAllText("result.xml") };
    }

    class XmlWriter : XmlTextWriter
    {
      public XmlWriter(string filename) : base(filename, Encoding.Default)
      {
        this.Formatting = System.Xml.Formatting.Indented;
      }
      public override void WriteString(string text)
      {
        DateTime d;
        if(DateTime.TryParse(text, out d))
          base.WriteString(d.ToShortDateString());
        else
          base.WriteString(text);
      }
    }
  }
}
```
