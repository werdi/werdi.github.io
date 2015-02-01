---
title: Как можно сделать цепь между элементами как в Multisim
tags: [c#, winforms, graphics]
src: https://social.msdn.microsoft.com/Forums/ru-RU/f6aae7cf-c217-482b-9140-e046fbad243b/-multisim?forum=fordesktopru
---
# Как можно сделать цепь между элементами как в Multisim
*> интересует именно как провести линии мишкой чтоб они били по 90градусов, так как на изображении*
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
      this.Size = new Size(500, 500);
      new Canvas { Dock = DockStyle.Fill, Parent = this };
    }
  }

  class Canvas : UserControl
  {
    public Canvas()
    {
      var img = new Bitmap(10, 10);
      using (var g = Graphics.FromImage(img))
      {
        g.Clear(Color.WhiteSmoke);
        img.SetPixel(0, 0, Color.Silver);
        img.SetPixel(9, 9, Color.Silver);
        img.SetPixel(9, 0, Color.Silver);
        img.SetPixel(0, 9, Color.Silver);
      }

      this.SetStyle(ControlStyles.UserPaint | ControlStyles.AllPaintingInWmPaint | ControlStyles.OptimizedDoubleBuffer, true);
      this.BackgroundImage = img;
    }

    protected override void OnMouseMove(MouseEventArgs e)
    {
      base.OnMouseMove(e);
      if (e.Button == MouseButtons.Left)
      {
        var cur = new Point(Node(e.X, 10), Node(e.Y, 10));
        this.Path = new GraphicsPath();
        var p1 = new Point(cur.X, this.Start.Y);
        this.Path.AddLine(this.Start, p1);
        this.Path.AddLine(p1, cur);
        this.Invalidate();
      }
      else
      {
        var a = new Point(Node(e.X, 10), Node(e.Y, 10));
        if (a != this.Active)
        {
          this.Active = a;
          this.Invalidate();
        }
      }
    }

    int Node(int p, int size)
    {
      var mp = ((p / size) * size);
      var dp = p - mp;
      return (dp <= size / 2) ? mp : mp + size;
    }

    protected override void OnMouseDown(MouseEventArgs e)
    {
      base.OnMouseDown(e);
      this.Start = this.Active;
      this.Path = null;
      this.Invalidate();
    }

    GraphicsPath Path;
    Point Active;
    Point Start;

    protected override void OnPaint(PaintEventArgs e)
    {
      base.OnPaint(e);
      if (this.Active.IsEmpty == false && Control.MouseButtons == MouseButtons.None)
      {
        var p = Point.Subtract(this.Active, new Size(5, 5));
        e.Graphics.DrawRectangle(Pens.Red, new Rectangle(p, new Size(9, 9)));
      }
      if(this.Path != null)
        e.Graphics.DrawPath(Pens.Red, this.Path);
    }
  }
}
```
*> почему я не могу разместить на этой сетке никакого изображения picturesbox?*

в конструкторе формы вместо: new Canvas { Dock = DockStyle.Fill, Parent = this };
укажите, например:
```c#
var c = new Canvas { Dock = DockStyle.Fill, Parent = this };
c.Controls.Add(new PictureBox() { Width=200, Height=200, Image = new Bitmap(@"d:\test\Photo.jpg") });
```
*> сделать чтоб изображение можно било на форме перемещать*

см. [здесь] (http://social.msdn.microsoft.com/Forums/en-US/programminglanguageru/thread/b676f743-0347-47cf-adea-d18235336a70/#c0a48075-c0b2-4121-a846-8bf73c85175a); в пример надо добавить следующий код: 
```c#
class Picture : Shape
{
  Image Image;
  Point Location;
  public override void Draw(Graphics g, bool active)
  {
    g.DrawImageUnscaled(this.Image, this.Location);
  }
  public override void Move(int dx, int dy)
  {
    this.Location.Offset(dx, dy);    
  }
  ...
}
```
*> как провести линии мишкой чтоб они били по 90градусов [...] подкинте идею как это реализовать!*

выше по теме есть [ответ] (http://social.msdn.microsoft.com/Forums/ru-RU/fordesktopru/thread/f6aae7cf-c217-482b-9140-e046fbad243b/#da7fe907-8650-4fc7-a933-4f369df7933e). 
для создания схем можно использовать Visio. см. [Host an Interactive Visio Drawing Surface in .NET Custom Clients] (http://msdn.microsoft.com/en-us/magazine/cc164043.aspx)
