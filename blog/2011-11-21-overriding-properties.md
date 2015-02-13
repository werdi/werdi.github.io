---
title: Переопределить свойство
tags: [c#, winforms, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/c6f9532c-fe0b-452d-b89e-c78ad8c8eb09/vs2010-?forum=fordesktopru
---
# Переопределить свойство
*> как переопределить шрифт (цвет, размер, стиль) по умолчанию в ячейке и/или строке Datagridview? И как переопределить цвет и фон в режиме редактирования?*
```c#
using System.Data;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable();
      dt.Columns.Add("Id", typeof(int));
      dt.Columns.Add("Value", typeof(string));
      for(int i=0; i < 5; i++) dt.Rows.Add(i, "v"+i);

      var dg = new DataGridView
      {
        Parent = this,
        Dock = DockStyle.Fill,
        AutoGenerateColumns = true,
        DataSource = dt
      };

      // стиль колонки
      var cs = dg.Columns[0].DefaultCellStyle;
      cs.BackColor = Color.Silver;
      cs.Font = new Font("verdana", 12);

      // стиль определенной ячейки
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("cell", (s, e) =>
        dg.Rows[0].Cells[1].Style.ApplyStyle(new DataGridViewCellStyle() { BackColor = Color.Red }));

      // стиль строки
      this.Menu.MenuItems.Add("row", (s, e) =>
      {
        foreach(DataGridViewCell c in dg.Rows[2].Cells)
          c.Style.ApplyStyle(new DataGridViewCellStyle() { BackColor = Color.Red });
      });

      // стиль редактируемой ячейки
      dg.EditingControlShowing += (s, e) => 
        e.CellStyle.BackColor = Color.Green;
    }
  }
}
```
*> тогда по умолчанию все DataGridView этого типа имеют параметры шрифта, такие как у font)
но не знаю, как это сделать для стиля редактируемой ячейки (какое свойство использовать)*

по-моему проще наследовать DataGridView и в конструкторе наследника указать 
значения свойств.
если переопределение свойств делается 'для защиты' от неправильного использования,
то 'неправильный' пользователь при желании сможет изменить любые значения, и приватные тоже, с помощью рефлексии.

*> Так я и делаю новый контрол, наследуя DataGridView, просто у меня много (~40) DataGridView - я и хочу по умолчанию задать свойства, кот. мне надо, чтобы не править потом у всех контролов.*

ниже пример наследования. в наследнике указываем свойства. используем контрол в 40 местах.
если в будущем надо поменять свойства у всех, то просто меняем один раз в конструкторе
WindowsFormsApplication2.DataGridView.
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      // контролы типа WindowsFormsApplication2.DataGridView
      new DataGridView { Parent = this, Dock = DockStyle.Fill };
      new DataGridView { Parent = this, Dock = DockStyle.Top };
    }
  }

  public class DataGridView : System.Windows.Forms.DataGridView
  {
    public DataGridView()
    {
      // здесь определяем значения свойств
    }
  }
}
```