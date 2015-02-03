---
title: OOBE и папки в ней
tags: [c#, win api, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b409d5fc-a4e9-4e89-98e7-8fce8a892a3b/oobe-?forum=fordesktopru
---
# OOBE и папки в ней
*> Существует в винде такая папка C:\Windows\System32\oobe  [...] папки есть, но как бы их нет*
```c#
using System.IO;
...
foreach(var fse in Directory.EnumerateFileSystemEntries(@"C:\Windows\System32\oobe"))
  System.Diagnostics.Trace.WriteLine(fse);
```
возвращает папки и файлы. проверил в Visual Studio 2010, "Run as administrator"

*> увы, этот код нашёл, лишь два файла и всё ту же папку, а файлов в папке намного больше. и да...*

проверьте следующий пример. 
сканирует папку через прямые вызовы Win API.
```c#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.IO;
using System.Runtime.InteropServices;
using System.Windows.Forms;
using Microsoft.Win32.SafeHandles;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(400, 500);
      var rtb = new RichTextBox { Parent = this, Dock = DockStyle.Fill };

      foreach(var data in ReadFolder(@"C:\Windows\System32\oobe\"))
        rtb.AppendText(data.FileName + " - " + data.FileAttributes + "\n");
    }

    IEnumerable<WIN32_FIND_DATA> ReadFolder(string path)
    {
      var data = new WIN32_FIND_DATA();
      using (var h = FindFirstFile(path.TrimEnd('\\', ' ') + "\\*", data))
      {
        if (h.IsInvalid) throw new Win32Exception();
        do yield return data; while (FindNextFile(h, data));
      }
    }

    [StructLayout(LayoutKind.Sequential)]
    class WIN32_FIND_DATA
    {
      public FileAttributes FileAttributes;
      public uint CreationTimeLow;
      public uint CreationTimeHigh;
      public uint LastAccessTimeLow;
      public uint LastAccessTimeHigh;
      public uint LastWriteTimeLow;
      public uint LastWriteTimeHigh;
      public int FileSizeHigh;
      public int FileSizeLow;
      public int Reserved0;
      public int Reserved1;
      [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 260)]
      public string FileName;
      [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 14)]
      public string AlternateFileName;
    }
    class SafeFindHandle : SafeHandleZeroOrMinusOneIsInvalid
    {
      [DllImport("kernel32.dll")]
      static extern bool FindClose(IntPtr handle);

      internal SafeFindHandle()
        : base(true)
      {
      }
      protected override bool ReleaseHandle()
      {
        return FindClose(base.handle);
      }
    }
    [DllImport("kernel32.dll", SetLastError = true)]
    static extern SafeFindHandle FindFirstFile(string fileName, [In, Out] WIN32_FIND_DATA data);
    [DllImport("kernel32.dll", SetLastError = true)]
    static extern bool FindNextFile(SafeFindHandle hndFindFile,
      [In, Out, MarshalAs(UnmanagedType.LPStruct)] WIN32_FIND_DATA lpFindFileData);
  }
}
```
