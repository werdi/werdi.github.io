---
title: Событие OnPaint WinForms
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/71fbf548-5446-4f60-9535-530616fefad7/-onpaint-winforms?forum=fordesktopru
---
# Событие OnPaint WinForms
*> если в этом PictureBox'е установить обработчик события OnPaint и в этом обработчике выполнить хотя бы одну элементарную операцию, касающуюся рисования, то PictureBox рисует лишь красный прямоугольник с диагоналями*

ниже пример с разными способами вывода графики в PictureBox и в Bitmap. графику лучше выводить в Bitmap.
```c#
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication0
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(300, 400);

      var bmp = new Bitmap(100, 100);
      using (var g = Graphics.FromImage(bmp))
        g.DrawPie(Pens.Red, 0, 0, bmp.Width - 1, bmp.Height - 1, 45, 90);

      new PictureBox { Parent = this, Dock = DockStyle.Fill, Image = bmp }
        .Paint += (s, e) =>
          {
            e.Graphics.DrawRectangle(Pens.Navy, 10, 10, 200, 150);
            e.Graphics.DrawEllipse(Pens.Green, 50, 50, 200, 100);
          };

      new MyPictureBox { Parent = this, Dock = DockStyle.Bottom, Height = 180, Image = bmp };
    }
  }
  class MyPictureBox : PictureBox
  {
    public MyPictureBox()
    {
      this.SetStyle(ControlStyles.UserPaint | ControlStyles.OptimizedDoubleBuffer | ControlStyles.AllPaintingInWmPaint, true);
    }
    protected override void OnPaint(PaintEventArgs e)
    {
      base.OnPaint(e);
      e.Graphics.DrawEllipse(Pens.Maroon, 20, 20, 90, 90);
    }
  }
}
```