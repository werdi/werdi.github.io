---
title: Сохранение сложной структуры в MySettings
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/37907ba9-d5a5-46f7-9713-b9ef877f55d7/-mysettings?forum=vsru
---
# Сохранение сложной структуры в MySettings
*> arraylist отобразить на другой форме в datagridview [...] решил воспользоваться mysettings [...] или как можно сделать это по другому?*

вместо ArrayList используйте DataTable и методы [WriteXml] (http://msdn.microsoft.com/ru-ru/library/system.data.datatable.writexml.aspx) и [ReadXml] (http://msdn.microsoft.com/ru-ru/library/fs0z9zxd.aspx)
```c#
using System.Data;
using System.Windows.Forms;
using System.IO;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable("point");
      dt.Columns.Add("X", typeof(int));
      dt.Columns.Add("Y", typeof(int));
      dt.Rows.Add(1, 1);
      dt.Rows.Add(2, 2);
      new DataGridView { Parent = this, Dock = DockStyle.Fill, DataSource = dt };

      var file = "test.xml";
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Save", delegate
      {
        dt.WriteXml(file, XmlWriteMode.WriteSchema);
      });
      this.Menu.MenuItems.Add("Load", delegate
      {
        if (File.Exists(file)) { dt.Clear(); dt.ReadXml(file); }
      });
      this.Menu.MenuItems.Add("New form", delegate { NewForm(file); });
    }

    void NewForm(string file)
    {
      var frm = new Form();
      var dt = new DataTable();
      dt.ReadXml(file);
      new DataGridView { Parent = frm, Dock = DockStyle.Fill, DataSource = dt };
      frm.Show();
    }
  }
}
```
*> компилятор выдал мне 4 синтаксические ошибки [...] Что я делаю не так?*

код DataTable table = new DataTable(); и т.д. поместите в конструктор Wizard_etalon() после вызов InitializeComponent().

*> как тогда мне сделать что бы по событию добовлять в dt строку?*
```c#
public partial class Form1 : Form
{
  DataTable dt;  // виден во всех методах Form1;

  public Form1()
  {
    dt = new DataTable("point");
    // ...
  }
  private void button1_Click(object sender, EventArgs e)
  {
    var dr = dt.NewRow();
    dr["X"] = 10;
    // ...
    dt.Rows.Add(dr);
  }
}
```
*> как сделать чтобы dt_etalon отобразился в уже существующей форме в существующих компонентах без создания новой формы.*

примерно так как у вас написано выше: dataGridView1.DataSource = dt_etalon; 
кроме грида есть другие компоненты на форме? какие?
опишите лучше саму задачу.

*> форма [...] меню [...] запускается мастер [...] происходит создания dt_etalon и файла text.xml при нажатии на кнопку финиш в компонент DataGridView должен залиться dt_etalon.*
```c#
using System.Data;
using System.IO;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Width = 600;
      var g = new DataGridView { Dock = DockStyle.Left, Width = 300, Parent = this };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("мастер", delegate
      {
        var file = "text.xml";
        File.Delete(file);

        var w = new Wizard(file);
        w.ShowDialog();

        if (File.Exists(file))
        {
          var dt = new DataTable();
          dt.ReadXml(file);
          g.DataSource = dt;
        }
      });
    }
  }

  class Wizard : Form
  {
    public Wizard(string file)
    {
      var dt = new DataTable("data");
      dt.Columns.Add("X", typeof(int));
      dt.Columns.Add("Y", typeof(int));
      dt.Rows.Add(1, 1);
      dt.Rows.Add(2, 2);
      new DataGridView { Dock = DockStyle.Fill, Width = 200, Parent = this, DataSource = dt };
      
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("финиш", delegate
      {
        dt.WriteXml(file, XmlWriteMode.WriteSchema);
        this.Close();
      });
    }
  }
}
```
