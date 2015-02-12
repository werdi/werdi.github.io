---
title: Mouse events like click button in c# console
tags: [c#, winforms, interop, console]
src: https://social.msdn.microsoft.com/Forums/ru-RU/e9e38863-30c3-4ca7-ab9f-c90bace89a7f?forum=csharpgeneral
---
# Mouse events like click button in c# console
*> I want to use mouse events like click button in c# console.*

call a SetConsoleMode function with ENABLE_MOUSE_INPUT and use ReadConsoleInput.
an example is here: [Reading Input Buffer Events](http://msdn.microsoft.com/en-us/library/windows/desktop/ms685035(v=vs.85).aspx) 

*> I tried but i couldn't do it .. :( can you create a basic console application,please ?*
 
```c#
using System;
using System.Linq;
using System.Runtime.InteropServices;

namespace ConsoleApplication1
{
	class Program
  {
    [STAThread]
    static void Main(string[] args)
    {
      var hStdin = GetStdHandle(-10); // STD_INPUT_HANDLE
      if (hStdin == new IntPtr(-1)) // INVALID_HANDLE_VALUE
          return;

      // ENABLE_WINDOW_INPUT | ENABLE_MOUSE_INPUT
      if (SetConsoleMode(hStdin, 0x0008 | 0x0010) == false) 
          return;

      var counter = 0;
      while (counter++ <= 10000)
      {
        int num;
        InputRecord record;
        if (ReadConsoleInput(hStdin, out record, 1, out num) == false) 
            break;

        switch(record.EventType)
        {
          case  EventType.MOUSE_EVENT:
            System.Diagnostics.Trace.WriteLine(ToString(record.mouse));
            break;

          case EventType.KEY_EVENT:
            System.Diagnostics.Trace.WriteLine(ToString(record.key));
            break;

          default:
            System.Diagnostics.Trace.WriteLine(record.EventType);
            break;
        }
      }
    }

    static string ToString(object o)
    {
      return String.Concat(
        o.GetType().Name,
        "={", String.Join("; ", o.GetType().GetFields().Select(f => f.GetValue(o))), "}");
    }

    public enum EventType : short
    {
      FOCUS_EVENT = 0x0010,
      KEY_EVENT = 0x0001,
      MENU_EVENT = 0x0008,
      MOUSE_EVENT = 0x0002
    }
    [StructLayout(LayoutKind.Explicit)]
    public struct InputRecord
    {
      [FieldOffset(0)]
      public EventType EventType;
      [FieldOffset(4)]
      public KeyEventRecord key;
      [FieldOffset(4)]
      public MouseEventRecord mouse;
    }
    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
    public struct KeyEventRecord
    {
      public bool keyDown;
      public short repeatCount;
      public short virtualKeyCode;
      public short virtualScanCode;
      public char uChar;
      public int controlKeyState;
    }
    [StructLayout(LayoutKind.Sequential)]
    public struct MouseEventRecord
    {
      public short X;
      public short Y;
      public int ButtonState;
      public int KeyState;
      public int EventFlags;
    }
    [DllImport("kernel32.dll", SetLastError = true)]
    static extern IntPtr GetStdHandle(int nStdHandle);
    [DllImport("kernel32.dll")]
    static extern bool SetConsoleMode(IntPtr hConsoleHandle, uint dwMode);
    [DllImport("kernel32.dll")]
    static extern bool GetConsoleMode(IntPtr hConsoleHandle, out uint lpMode);
    [DllImport("kernel32.dll", EntryPoint = "ReadConsoleInput", CharSet = CharSet.Unicode)]
    extern static bool ReadConsoleInput(IntPtr handle, out InputRecord record, int len, out int num);
  }
}
```   