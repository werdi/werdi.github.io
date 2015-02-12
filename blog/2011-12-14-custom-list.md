---
title: Custom list
tags: [c#, winforms, linq, imaging]
src: https://social.msdn.microsoft.com/Forums/ru-RU/7271a006-7d4d-4dbb-9603-7c0c3542caf3/custom-list?forum=csharpgeneral
---
# Custom list
*> Im creating a C# winforms app and I need to create a custom list. [...] one of the items is an image and some data. the left should have the image and on the right should be the data about it. lets say, artist, year, price style, etc...*

if you want to output data as list without any selector's logic, then try using the following code:
```c#
using System;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication6
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = LoadData();
      var cs = dt.Columns.OfType<DataColumn>()
                .Except(new[] { dt.Columns["Id"], dt.Columns["Image"] });
      foreach (DataRow row in dt.Rows)
      {
        var p = new Panel { Parent = this, Dock = DockStyle.Top, Height = 100, 
                  Padding = new Padding(2) };
        var r = new Panel { Parent = p, Dock = DockStyle.Fill, Height = 100 };
        new PictureBox { Parent = p, Dock = DockStyle.Left, Width = 100, 
           Image = (Image) row["Image"] };
        foreach (var c in cs)
        {
          new Label
          {
            Padding = new Padding(5, 3, 3, 3),
            Parent = r,
            Dock = DockStyle.Top,
            Height = 20,
            TextAlign = ContentAlignment.MiddleLeft,
            Text = c.ColumnName + ": " + row[c.ColumnName].ToString()
          };
        }
      }
    }

    private DataTable LoadData()
    {
      var dt = new DataTable();
      dt.Columns.Add("Id", typeof(int));
      dt.Columns.Add("Title", typeof(string));
      dt.Columns.Add("Artist", typeof(string));
      dt.Columns.Add("Year", typeof(int));
      dt.Columns.Add("Image", typeof(Image));

      // create image; for test
      Func<Color, Image> img = c => 
        {
          Bitmap ret = new Bitmap(100, 100);
          using(var g = Graphics.FromImage(ret))
          using(var p = new Pen(c))
            g.DrawEllipse(p, new Rectangle(15,15,75,75));
          return ret;
        };

      // test data
      dt.Rows.Add(0, "t1", "a1", 2001, img(Color.Red));
      dt.Rows.Add(1, "t2", "a2", 2002, img(Color.Green));
      return dt;
    }
  }
}
```