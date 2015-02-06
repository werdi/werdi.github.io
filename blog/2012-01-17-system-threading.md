---
title: System.Threading Threads?
tags: [c#, winforms, threading]
src: https://social.msdn.microsoft.com/Forums/ru-RU/f908a989-63d7-4a8c-ac67-1b9c41aaa6cc/systemthreading-threads?forum=csharpgeneral
---
# System.Threading Threads?
*> I'm trying to create a game that has a lot of things going on at once [...]  So I'm thinking splitting everything up to different threads [...] I'm wondering if starting a new thread will allow me to call a method that needs to be updated*

try using the following example
```c#
using System;
using System.ComponentModel;
using System.Drawing;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.SetStyle(ControlStyles.AllPaintingInWmPaint
        | ControlStyles.ResizeRedraw | ControlStyles.UserPaint
        | ControlStyles.OptimizedDoubleBuffer, true);

      var lbl = new Label { Parent = this, Dock = DockStyle.Top };

      var cts = new CancellationTokenSource();
      this.Shown += (s, e) => Run(cts.Token, str => lbl.Text = str);
      this.FormClosing += (s, e) => cts.Cancel();
    }

    void Run(CancellationToken ct, Action<string> callback)
    {
      var ao = AsyncOperationManager.CreateOperation(null);
      int a = 200;
      int count = 0;
      _Event = new ManualResetEvent(false);
      ThreadPool.QueueUserWorkItem(s =>
      {
        while (ct.IsCancellationRequested == false)
        {
          _Event.Reset();

          // post data to the UI thread
          ao.Post(str => callback("info: " + str), count++);

          var bmp = new Bitmap(this.ClientRectangle.Width, this.ClientRectangle.Height);
          using (var g = Graphics.FromImage(bmp))
          {
            // draw something here
            g.Clear(Color.FromArgb(a, Color.Red));
          }
          
          a -= 3;
          if(a < 50) a = 200;

          _Image = bmp;
          this.Invalidate();
          _Event.WaitOne();
          Thread.Sleep(15);
        }
      });
    }

    ManualResetEvent _Event;
    Bitmap _Image;

    protected override void OnPaint(PaintEventArgs e)
    {
      base.OnPaint(e);
      if (_Image != null)
        e.Graphics.DrawImageUnscaled(_Image, 0, 0);

      _Event.Set();
    }
  }
}
```
