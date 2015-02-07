---
title: IDownloadManager (плагин для Internet Explorer), Custom download manager
tags: [c#, ie, com, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/6de59cee-5b7d-4030-9301-97defdbae513/idownloadmanager-internet-explorer-custom-download-manager?forum=programminglanguageru
---
# IDownloadManager (плагин для Internet Explorer), Custom download manager
*> IDownloadManager [...]  Написал реализацию данного интерфейса на шарпе. [...] метод не вызывается (не доходит до этого). [...] [DispId(1)] void Download(*
```c#
using System;
using System.Runtime.InteropServices;
using System.Runtime.InteropServices.ComTypes;
...
[ComVisible(true), ComImport,
Guid("988934A4-064B-11D3-BB80-00104B35E7F9"),
InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
public interface IDownloadManager
{
  [return: MarshalAs(UnmanagedType.I4)]
  [PreserveSig]
  int Download(
    [In, MarshalAs(UnmanagedType.Interface)] IMoniker pmk,
    [In, MarshalAs(UnmanagedType.Interface)] IBindCtx pbc,
    [In, MarshalAs(UnmanagedType.U4)] UInt32 dwBindVerb,
    [In] Int32 grfBINDF,
    ref IntPtr pBindInfo,
    [In, MarshalAs(UnmanagedType.LPWStr)] string pszHeaders,
    [In, MarshalAs(UnmanagedType.LPWStr)] string pszRedir,
    [In, MarshalAs(UnmanagedType.U4)] UInt32 uiCP
    );
}
```
