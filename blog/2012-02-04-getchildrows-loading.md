---
title: Two questions about GetChildRows
tags: [c#, winforms, datagridview]
src: https://social.msdn.microsoft.com/Forums/ru-RU/2f835c95-c34e-477a-9182-6dfb0fa18be5/two-questions-about-getchildrows?forum=winformsdatacontrols
---
# Two questions about GetChildRows
*> dataGridView1.DataSource = dt1; [...] dataGridView2.DataSource = ds.Tables[0].Rows[e.RowIndex].GetChildRows("newrelation");*

if i understood correctly you want to create master-detail interface.
if so, then take a look to the following example.
```c#
using System.Windows.Forms;
using System.Data;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(900, 400);

      // dataset
      var dt1 = new DataTable("master");
      dt1.Columns.Add("Id", typeof(int));
      dt1.Columns.Add("Name", typeof(string));
      
      var dt2 = new DataTable("details");
      dt2.Columns.Add("Id", typeof(int));
      dt2.Columns.Add("Name", typeof(string));

      var ds = new DataSet();
      ds.Tables.AddRange(new[] { dt1, dt2 });
      ds.Relations.Add("master_details", dt1.Columns["Id"], dt2.Columns["Id"]);

      // test data
      for(int i=0; i < 5; i++) 
        dt1.Rows.Add(i, "n"+i);
      foreach(DataRow r in dt1.Rows)
      {
        var id = r["Id"];
        for(int i=0; i < 3; i++)
          dt2.Rows.Add(id, "n"+i);
      }

      // controls
      var dg = new DataGrid { Parent = this, Dock = DockStyle.Fill, DataSource = ds };
      this.Shown += (s, e) => 
      {
        dg.Focus();
        SendKeys.SendWait("%({DOWN})");
      };
      new DataGridView { Dock = DockStyle.Left, Width = 300, Parent = this, DataSource = ds, DataMember = "master.master_details" };
      new DataGridView { Dock = DockStyle.Left, Width = 300, Parent = this, DataSource = ds, DataMember = "master" };
    }
  }
}
```
