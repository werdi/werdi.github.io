---
title: Problem with owner drawn tabs
tags: [c#, winforms, imaging]
src: https://social.msdn.microsoft.com/Forums/ru-RU/ef09f8b3-7445-4ffa-be9b-7ae7a93b1167/problem-with-owner-drawn-tabs?forum=csharplanguage
---
# Problem with owner drawn tabs
*> I have a tabControl object [...] OwnerDrawn = true. [...] I would like it so that the tab BG color/image (app setting) changes [...] So, if the other client sends StatusChanged(Status) to mine, it will update the theme of the individual tab*
```c#
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    class Info
    {
      public string Title;
      public Color Color;
    }

    public Form1()
    {
      var tabs = new Info []
      {
        new Info { Title = "Red", Color = Color.Red },
        new Info { Title = "Green", Color = Color.Green },
        new Info { Title = "Blue", Color = Color.Blue },
      };

      var tc = new TabControl { Parent = this, Dock = DockStyle.Fill };
      foreach (var tab in tabs)
        tc.TabPages.Add(new TabPage(tab.Title)
        {
            // BackColor = tab.Color
        });
      tc.DrawMode = TabDrawMode.OwnerDrawFixed;
      tc.DrawItem += (s, e) =>
      {
        var t = tabs[e.Index];
        using (var b = new SolidBrush(t.Color))
        using (var f = new StringFormat
        {
          Alignment = StringAlignment.Center,
          LineAlignment = StringAlignment.Center
        })
        {
          e.Graphics.FillRectangle(b, e.Bounds);
          e.Graphics.DrawString(t.Title, this.Font, Brushes.Wheat, e.Bounds, f);
        }
      };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Test", (s, e) =>
      {
        tabs[1].Color = Color.Violet;
        tc.Invalidate();
      });
    }
  }
}
```
take a look at the following example. 
it paints a different background color and image on each tabpage.
```c#
using System.Data;
using System.Drawing;
using System.IO;
using System.Windows.Forms;
using System;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    class Info
    {
      public string Title;
      public Color Color;
      public Image Image;
    }

    public Form1()
    {
      Func<Color, Image> getImage = c =>    // for test
      {
        Bitmap b = new Bitmap(100, 10);
        using (var g = Graphics.FromImage(b)) g.Clear(c);
        return b;
      };

      var tabs = new Info[]
      {
        new Info { Title = "Red", Color = Color.Red, Image=getImage(Color.Silver) },
        new Info { Title = "Green", Color = Color.Green, Image=getImage(Color.Navy) },
        new Info { Title = "Blue", Color = Color.Blue, Image=getImage(Color.Green) },
      };

      var tc = new TabControl { Parent = this, Dock = DockStyle.Fill };
      foreach (var tab in tabs)
        tc.TabPages.Add(new TabPage(tab.Title)
        {
          BackColor = tab.Color
        });
      tc.DrawMode = TabDrawMode.OwnerDrawFixed;
      tc.DrawItem += (s, e) =>
      {
        var t = tabs[e.Index];
        using (var b = new SolidBrush(t.Color))
        using (var f = new StringFormat
        {
          Alignment = StringAlignment.Center,
          LineAlignment = StringAlignment.Center
        })
        {
          e.Graphics.FillRectangle(b, e.Bounds);
          e.Graphics.DrawString(t.Title, this.Font, Brushes.Wheat, e.Bounds, f);
        }
      };

      foreach (TabPage tab in tc.TabPages)
      {
        tab.Paint += (s, e) => 
        {
          var i = tc.TabPages.IndexOf((TabPage)s);
          System.Diagnostics.Trace.WriteLine(i);
          e.Graphics.DrawImageUnscaled(tabs[i].Image, new Point(0, 10));
        };
      }

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Test", (s, e) =>
      {
        tabs[1].Color = Color.Violet;
        tc.Invalidate();
      });
    }
  }
}
```