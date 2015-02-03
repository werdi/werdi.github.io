---
title: Get list of installed DBMS in a computer using c#
tags: [c#, winforms, wmi]
src: https://social.msdn.microsoft.com/Forums/ru-RU/dc7ce512-fbc0-43aa-82db-fb0f1e321ede/get-list-of-installed-dbms-in-a-computer-using-c?forum=netfxbcl
---
# Get list of installed DBMS in a computer using c#
*> is there any way (or API) to use to get a list of all installed DBMS in a computer using C# ?I heared WMI can get informations about the system , but can it be used to get installed DBMS ?*

if you know a vendor's name and the caption of the product you can filter the Win32_Product list.
below is an example.
```c#
using System.Collections.Generic;
using System.Linq;
using System.Management;
using System.Windows.Forms;
using System;
using System.Threading.Tasks;
using System.Threading;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(800, 500);
      var rtb = new RichTextBox { 
        Parent = this, Dock = DockStyle.Fill, Text = "Searching ... (it takes too long time)" };
      Action<string> notify = str => rtb.AppendText("\n" + str);
      var cts = new CancellationTokenSource();
      var tsk = Task.Factory.StartNew(() =>
      {
        var res = from mo in WmiSearch("SELECT * FROM Win32_Product")
                    select new
                    {
                      Caption = mo.GetPropertyValue("Caption"),
                      Description = mo.GetPropertyValue("Description"),
                      Vendor = mo.GetPropertyValue("Vendor")
                    };
        foreach (var p in res.Where(p => p.Caption.ToString().Contains("SQL")))
          this.BeginInvoke(notify, p.Vendor + ": " + p.Caption);
      });

      this.FormClosing += (s, e) =>
      {
        if(tsk.IsCompleted == false)
        {
          notify("Wait...");
          e.Cancel = true;
        }
      };
    }
    static IEnumerable<ManagementObject> WmiSearch(string query)
    {
      var searcher = new ManagementObjectSearcher(query);
      foreach (ManagementBaseObject mbo in searcher.Get())
      {
        var mo = (mbo as ManagementObject);
        if (mo != null)
          yield return mo;
      }
    }
  }
}
```
