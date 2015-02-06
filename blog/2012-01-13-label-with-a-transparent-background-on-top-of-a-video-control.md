---
title: Is there any way to create a label with a transparent background on top of a video control?
tags: [c#, interop, winforms, win api]
src: https://social.msdn.microsoft.com/Forums/ru-RU/d4e92dd4-01d7-45b5-ba9e-a623658e1bf9/is-there-any-way-to-create-a-label-with-a-transparent-background-on-top-of-a-video-control?forum=winforms
---
# Is there any way to create a label with a transparent background on top of a video control?
*> is there any way to superimpose a label with a transparent background on top of it?*

try using a following [example] (http://social.msdn.microsoft.com/Forums/en-US/wpf/thread/c32889d3-effa-4b82-b179-76489ffa9f7d#408c8a8a-991f-4a75-a826-7c0feaeda830) (creates transparent window invisible for mouse)

another approach is to create Layered Window.
take a look at the article "[Layered Windows] (http://msdn.microsoft.com/en-us/library/ms997507.aspx)" and [OSFeature.LayeredWindows] (http://msdn.microsoft.com/en-us/library/system.windows.forms.osfeature.layeredwindows.aspx) Field.

*> I should have mentioned this is a conventional Winforms (4.0) app, not WPF.*

you can use System.Windows.Forms.Integration.ElementHost, MediaElement and Label.
below is an example.
```c#
using System;
using System.Windows.Forms;
using System.Windows.Forms.Integration;
using Wpf = System.Windows.Controls;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var eh = new ElementHost { Dock = DockStyle.Fill, Parent = this };
      var me = new Wpf.MediaElement { Source = new Uri(@"z:\test.wmv") };
      var lbl = new Wpf.Label { Content = "Hello", FontSize = 50 };
      var grd = new Wpf.Grid();
      grd.Children.Add(me);
      grd.Children.Add(lbl);
      eh.Child = grd;
    }
  }
}
```
*> All the methods I've tried so far have led to opaque backgrounds or horrible flickering. I suspect that putting a transparent background control on a video stream would be an excessive load on the system.*

try using the following example
```c#
using System;
using System.Drawing;
using System.Drawing.Imaging;
using System.Runtime.InteropServices;
using System.Windows.Forms;
using AxWMPLib;   // Windows Media Player

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new Size(500, 300);
      this.TopMost = true;
      Func<Bitmap> bmp = () =>
      {
        var ret = new Bitmap(450, 200);
        using (var g = Graphics.FromImage(ret))
          g.DrawString("Hi", new Font("verdana", 90), Brushes.Red, 0, 0);
        return ret;
      };
      var lw = new LayeredWindow(bmp)
      {
        Owner = this,
        Location = new Point(this.Left+20, this.Top+50)
      };
      this.LocationChanged += (s, e) =>
        lw.Location = new Point(this.Left+20, this.Top+50);
      lw.Show();

      var mp = new AxWindowsMediaPlayer { Parent = this, Dock = DockStyle.Fill };
      mp.HandleCreated += (s, e) =>
        mp.URL = 
          "http://download.microsoft.com/download/6/1/D" + 
          "/61DF31A6-C45C-41CD-9E5F-568FE5E7D23D" + 
          "/HDI_ITPro_MSDN_winvideo_MVC_3_Quick_Hits_Razor_View_Engine.wmv";
    }
  }

  public class LayeredWindow : Form
  {
    public LayeredWindow(Func<Bitmap> bmp)
    {
      this.TopMost = true;
      this.ShowInTaskbar = false;
      this.Shown += (s, e) => this.UpdateLayered(bmp());
    }
    protected override CreateParams CreateParams
    {
      get
      {
        var p = base.CreateParams;
        p.ExStyle |= 0x80000;  // WS_EX_LAYERED
        return p;
      }
    }
    void UpdateLayered(Bitmap bmp)
    {
      if (bmp.PixelFormat != PixelFormat.Format32bppArgb)
        throw new ArgumentException("32bpp with alpha-channel required");

      IntPtr screenDc = GetDC(new HandleRef());
      IntPtr memDc = CreateCompatibleDC(new HandleRef(null, screenDc));

      IntPtr hBmp = bmp.GetHbitmap(Color.FromArgb(0));
      SelectObject(new HandleRef(null, memDc), hBmp);

      this.Size = bmp.Size;

      var blendFunc = new BLENDFUNCTION
      {
        BlendOp = 0,
        BlendFlags = 0,
        SourceConstantAlpha = 0xff,
        AlphaFormat = 1
      };
      bool flag = UpdateLayeredWindow(
        this.Handle,
        screenDc,
        new POINT { X = this.Left, Y = this.Top },   // new location
        new POINT { X = this.Width, Y = this.Height },    // new size
        memDc,
        new POINT { X = 0, Y = 0 },  // source location
        0,
        ref blendFunc,
        2);
      ReleaseDC(new HandleRef(), new HandleRef(null, memDc));
      ReleaseDC(new HandleRef(), new HandleRef(null, screenDc));
    }
  
    [DllImport("user32.dll")]
    static extern bool UpdateLayeredWindow(IntPtr hwnd, IntPtr hdcDst, POINT pptDst, 
      POINT pSizeDst, IntPtr hdcSrc, POINT pptSrc, int crKey, ref BLENDFUNCTION pBlend, 
      int dwFlags);
    [StructLayout(LayoutKind.Sequential)]
    struct BLENDFUNCTION
    {
      public byte BlendOp;
      public byte BlendFlags;
      public byte SourceConstantAlpha;
      public byte AlphaFormat;
    }
    [DllImport("gdi32.dll")] static extern IntPtr SelectObject(HandleRef hdc, IntPtr obj);
    [DllImport("gdi32.dll")] static extern IntPtr CreateCompatibleDC(HandleRef hDC);
    [DllImport("user32.dll")] static extern IntPtr GetDC(HandleRef hWnd);
    [DllImport("user32.dll")] static extern int ReleaseDC(HandleRef hWnd, HandleRef hDC);
    [StructLayout(LayoutKind.Sequential)] public class POINT { public int X; public int Y; }
  }
}
```
p.s.
instead of AxWindowsMediaPlaye above, you can use [HTML5 player] (http://social.msdn.microsoft.com/Forums/en-us/wpf/thread/87e41b0d-145d-4438-958e-c8b3a0a969d3/#f64a413c-9d3a-4fef-88cf-27db4652e810)

*> why is it not moving with the rest of the form?*

maybe you have missed to bind a forms.
take a close note at the following line in the code above:
```c#
this.LocationChanged += (s, e) =>
  lw.Location = new Point(this.Left+20, this.Top+50);
```
