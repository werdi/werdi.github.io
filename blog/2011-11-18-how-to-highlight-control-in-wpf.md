---
title: How to highlight control in WPF
tags: [c#, xaml, wpf, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/82a323dd-5d87-43b3-9d22-56b2bfcdc863/how-to-highlight-control-in-wpf?forum=wpf
---
# How to highlight control in WPF

*> How can I highlight a control (a boarder of the control like the spy++ do) in my WPF application in real time?*

first you need to get window under mouse pointer.
then create transparent window with solid border.
an example of transparent window is [here](http://social.msdn.microsoft.com/Forums/en-US/wpf/thread/c32889d3-effa-4b82-b179-76489ffa9f7d#408c8a8a-991f-4a75-a826-7c0feaeda830).

*> externally, if I have the windowHandle (I can get the window under the mouse)*

below is an example, it works like a spy++
```xaml
<Window x:Class="WpfApplication0.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  WindowStyle="ToolWindow"
  FontSize="16"
  Topmost="True"
  Title="MainWindow" Width="300" Height="100">
  <TextBox x:Name="tb" />
</Window>
```
```c# 
using System;
using System.Runtime.InteropServices;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Interop;
using System.Windows.Media;

namespace WpfApplication0
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      TransparentWindow tw = null;
      Rect last = Rect.Empty;
      var dt = new System.Timers.Timer(100);
      dt.Elapsed += (s, e) =>
      {
        dt.Stop();
        var hwnd = User32.GetCursorHwnd();
        var bounds = User32.GetBounds(hwnd);
        if (bounds != last)
        {
          this.Dispatcher.BeginInvoke(new Action(() => 
          {
            tb.Text = bounds.ToString();

            if(tw != null) tw.Close();
            tw = new TransparentWindow(bounds);
          }));
        }
        last = bounds;
        dt.Start();
      };
      this.Loaded += (s, e) => dt.Start();
      this.Closing += (s, e) => System.Windows.Application.Current.Shutdown();
    }
  }

  class TransparentWindow : Window
  {
    public TransparentWindow(Rect b)
    {
      this.Width = b.Width;
      this.Height = b.Height;
      this.Left = b.Left;
      this.Top = b.Top;
      this.Background = Brushes.Transparent;
      this.ShowInTaskbar = false;
      this.AllowsTransparency = true;
      this.WindowStyle = System.Windows.WindowStyle.None;
      this.Topmost = true;
      this.WindowStartupLocation = System.Windows.WindowStartupLocation.Manual;
      MakeTransparent();
      this.Content = new Border { BorderBrush = Brushes.Red, BorderThickness = new Thickness(2) };
      this.Show();
    }

    public void MakeTransparent()
    {
      var hwnd = new WindowInteropHelper(this).Handle;
      int GWL_EXSTYLE = -20;
      var es = GetWindowLong(hwnd, GWL_EXSTYLE);
      SetWindowLong(hwnd, GWL_EXSTYLE, es | 0x00000020 /*WS_EX_TRANSPARENT*/);
    }

    [DllImport("user32.dll")]
    static extern int SetWindowLong(IntPtr hwnd, int index, int newStyle);
    [DllImport("user32.dll")]
    static extern int GetWindowLong(IntPtr hwnd, int index);
    [DllImport("user32")]
    static extern bool SetWindowPos(IntPtr hWnd, IntPtr hWndInsertAfter, int x, int y, int cx, int cy, uint uFlags);
  }
  
  public static class User32
  {
    [StructLayout(LayoutKind.Sequential)]
    struct Cursor
    {
      public Int32 X;
      public Int32 Y;
    }

    [StructLayout(LayoutKind.Sequential)]
    struct RECT
    {
      public Int32 Left;
      public Int32 Top;
      public Int32 Right;
      public Int32 Bottom;
    }

    [DllImport("user32.dll")]
    static extern IntPtr WindowFromPoint(Cursor point);

    [DllImport("user32.dll")]
    static extern bool GetCursorPos(ref Cursor pt);

    public static IntPtr GetCursorHwnd()
    {
      Cursor p = new Cursor();
      GetCursorPos(ref p);
      return WindowFromPoint(p);
    }

    public static Rect GetBounds(IntPtr hwnd)
    {
      RECT res;
      GetWindowRect(hwnd, out res);
      return new Rect(res.Left, res.Top, res.Right - res.Left, res.Bottom - res.Top);
    }

    [DllImport("user32.dll")]
    static extern bool GetWindowRect(IntPtr hwnd, out RECT lpRect);
  }
}
```