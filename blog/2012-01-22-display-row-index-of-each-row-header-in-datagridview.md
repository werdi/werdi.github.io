---
title: Display row index of each row header in dataGridView
tags: [c#, winforms, graphics]
src: https://social.msdn.microsoft.com/Forums/ru-RU/cd6af638-2674-476a-b481-83865ba0e26b/display-row-index-of-each-row-header-in-datagridview?forum=csharpgeneral
---
# Display row index of each row header in dataGridView
*> I've been trying to get my dataGridView to display row index on the rowheader*
```c#
using System.Data;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable();
      dt.Columns.Add("id", typeof(int));
      for (int i = 0; i < 5; i++) dt.Rows.Add(i);

      var dg = new DataGridView { 
        Dock = DockStyle.Fill, Parent = this, DataSource = dt, RowHeadersWidth = 40 };
      dg.RowPostPaint += (s, e) =>
      {
        var sf = new StringFormat { 
          LineAlignment = StringAlignment.Center, Alignment = StringAlignment.Near};
        var cr = e.RowBounds;
        cr.Offset(20, 0);
        e.Graphics.DrawString(e.RowIndex.ToString(), this.Font, Brushes.Red, cr, sf);
      };
    }
  }
}
```
*> I'd prefer having a separate private void for the RowPostPaint event, as the one I've shown above. What's the best way to do that?*

instead of dg.RowPostPaint += (s, e) => { ... }
write the following:
```c#
dg.RowPostPaint += dataGridView1_RowPostPaint;
...
void dataGridView1_RowPostPaint(object sender, DataGridViewRowPostPaintEventArgs e)
{
  ...
}
```
