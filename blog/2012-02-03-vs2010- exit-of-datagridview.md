---
title: VS2010 Выйти из DataGridView
tags: [c#, winforms, datagridview]
src: https://social.msdn.microsoft.com/Forums/ru-RU/2b7a6ae9-7bd8-4c1e-a8e1-4118db8c79f5/vs2010-datagridview?forum=fordesktopru
---
# VS2010 Выйти из DataGridView
*> выйти из DataGridView, если я нахожусь в последней ячейке [...] и перейти (например, правой стрелкой) на следующий по TabIndex контрол [...] в первой ячейке DataGridView [...] перейти на предыдущий контрол левой стрелкой?*
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Width = 400;
      new DataGridView
      {
        Dock = DockStyle.Fill,
        AllowUserToAddRows = false,
        Parent = this,
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill,
        DataSource = new[] { new { Id = 1, Text = "text" } }
      };

      new RichTextBox { Parent = this, Dock = DockStyle.Right, Width = 100 };
      new RichTextBox { Parent = this, Dock = DockStyle.Left, Width = 100 };
    }
}

class DataGridView : System.Windows.Forms.DataGridView
{
  bool FirstCell()
  {
    return this.CurrentCell.ColumnIndex == 0 && this.CurrentCell.RowIndex == 0;
  }
  bool LastCell()
  {
    return this.CurrentCell.ColumnIndex == this.ColumnCount - 1 
      && this.CurrentCell.RowIndex == this.RowCount - 1;
  }
  protected override bool ProcessDataGridViewKey(KeyEventArgs e)
  {
      if (LastCell() && (e.KeyCode == Keys.Enter || e.KeyCode == Keys.Right))
      {
        SendKeys.Send("{TAB}");
        return e.Handled = true;
      }
      if (FirstCell() && e.KeyCode == Keys.Left)
      {
        SendKeys.Send("+({TAB})");
        return e.Handled = true;
      }
      return base.ProcessDataGridViewKey(e);
    }
  }
}
```
