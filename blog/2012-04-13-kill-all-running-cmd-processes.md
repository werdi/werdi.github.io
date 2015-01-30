---
title: Kill all running cmd processes
tags: [c#, wmi, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/7c158a4b-ed53-4015-8f6a-5e8dc4994244/kill-all-running-cmd-processes?forum=csharplanguage
---
# Kill all running cmd processes
*> My program runs 10 cmd processes, and i want to add a stop button, so when you press it, it kills all 10 cmd processes.* 

this is possible with WMI. try using the following code:
```c#
using System.Diagnostics;
using System.Management;   // System.Management.dll
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    void Button_Click(object sender, System.EventArgs e)
    {
      TerminateChildProcess(Process.GetCurrentProcess().Id);
    }

    static void TerminateChildProcess(int parentId)
    {
      var mos = new ManagementObjectSearcher(
        string.Format("SELECT * FROM Win32_Process WHERE ParentProcessId = {0}", parentId));
      foreach (ManagementBaseObject mbo in mos.Get())
      {
        var p = mbo as ManagementObject;
        if (p == null) continue;
        uint res = (uint)p.InvokeMethod("Terminate", new object[0]);
        if (res != 0)
        {
          // handle error
        }
      }
    }
  }
}
```

you can find  the other filter criteria in the [Win32_Process] (http://msdn.microsoft.com/en-us/library/windows/desktop/aa394372(v=vs.85).aspx)
