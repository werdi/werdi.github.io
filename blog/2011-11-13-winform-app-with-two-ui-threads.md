---
title: How to create a winform app with two UI threads
tags: [c#, winforms, threading]
src: https://social.msdn.microsoft.com/Forums/ru-RU/3f0e7794-8671-47c4-aa9a-3bd1f85c9963/how-to-create-a-winform-app-with-two-ui-threads?forum=winforms
---
# How to create a winform app with two UI threads
*> the main form, and the other one will be for another form that behaves kind of like a splash screen.*

below is an example of one of the approaches for creating a mainform with splashscreen
```c#
using System;
using System.Data;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication0
{
  static class Program
  {
    [STAThread]
    static void Main()
    {
      Application.EnableVisualStyles();
      Application.SetCompatibleTextRenderingDefault(false);

      var dt = new DataTable();
      dt.Columns.Add("id", typeof(int));
      dt.Columns.Add("text", typeof(string));

      var sf = new Splashscreen(dt);
      sf.ShowDialog();

      Application.Run(new MainForm(dt));
    }
  }

  public partial class MainForm : Form
  {
    public MainForm(DataTable dt)
    {
      this.StartPosition = FormStartPosition.CenterScreen;
      this.Size = new Size(800, 600);
      new DataGridView { Parent = this, Dock = DockStyle.Fill, DataSource = dt };
    }
  }

  class Splashscreen : Form
  {
    public Splashscreen(DataTable dt)
    {
      this.StartPosition = FormStartPosition.CenterScreen;
      this.ShowInTaskbar = false;
      this.TopMost = true;
      this.ControlBox = false;
      this.FormBorderStyle = FormBorderStyle.FixedSingle;

      var lbl = new Label { Parent = this, Dock = DockStyle.Fill, Text = "Loading" };
      var t = new System.Windows.Forms.Timer { Interval = 100 };
      var i = 0;
      t.Tick += (s, e) =>
      {
        lbl.Text += ".";
        dt.Rows.Add(i++, "text" + DateTime.Now.Millisecond);
        if(i == 20)
        {
          this.Close();
          t.Dispose();
        }
      };
      t.Start();
    }
  }
}
```