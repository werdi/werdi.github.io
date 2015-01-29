---
title: C# Пересечение фигур
tags: [c#, region, graphics, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9ac581b3-5379-4e72-a36b-1832ac528da5/c-?forum=programminglanguageru
---
# C# Пересечение фигур
*> нужно проверить фигуры на пересечение*
см. [Region] (http://msdn.microsoft.com/ru-ru/library/1w2k0384.aspx)
пример вывода пересечения двух фигур:
```c#
using System.Drawing;
using System.Drawing.Drawing2D;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var p1 = new GraphicsPath();
      p1.AddRectangle(new Rectangle(30, 30, 100, 100));
      var p2 = new GraphicsPath();
      p2.AddEllipse(new Rectangle(5, 5, 50, 50));

      var r = new Region(p2);
      r.Intersect(p1);

      var img = new Bitmap(200, 200);
      using(var g = Graphics.FromImage(img))
      {
        g.FillRegion(Brushes.Red, r);
        g.DrawPath(Pens.Silver, p1);
        g.DrawPath(Pens.Silver, p2);
      }
      new PictureBox { Parent = this, Dock = DockStyle.Fill, Image = img };
    }
  }
}
```
